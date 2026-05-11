# 05. 100M-row Postgres schema migration with no downtime

**Prompt:**
> "I need to migrate a 100M-row Postgres table to a new schema without downtime. How should I approach this?"

---

## `/grill-me` output

Before we touch migration mechanics, the foundational question: **what does "no downtime" actually mean for this table's consumers?** Are you targeting (a) zero unavailability for reads and writes throughout, (b) a brief read-only window is acceptable, or (c) a sub-second write pause during cutover is acceptable?

My recommendation: **(c) — allow a sub-second write pause at cutover.** True zero-pause migrations require dual-writes plus reconciliation, which roughly doubles the engineering effort and risk surface. A sub-second pause (achieved via a transactional rename or a brief lock during the final sync) is invisible to almost all real-world consumers, dramatically simpler, and is what "online migration" tools (pg_repack, pgroll, Stripe/GitHub's published playbooks) actually deliver. Reserve (a) only if you have a specific SLA that forbids any write pause.

Which one applies to you?

---

## `/cook-me` output

**`Q1 of ~8.`**
> What's forcing the migration — is the new schema a backwards-compatible additive change, or a breaking restructure (renames, type changes, splits)?

**`Candidates`** _(↓ most to least promising)_

`1.` **Breaking restructure**<br>
Columns renamed, split, retyped, or moved across tables; old and new shapes can't coexist behind one view.<br>
_Dual-writes and backfill become unavoidable; the migration tool choice hinges entirely on this answer._<br>
`Leads to →` pgroll, pg-osc, Debezium CDC (+ more)

`2.` **Additive-only**<br>
New nullable columns or tables; old code keeps working untouched against the same rows.<br>
_Backfill is the only real risk; you can skip orchestration tooling and ship a plain migration._<br>
`Leads to →` plain DDL, batched backfill

`3.` **Physical reorg**<br>
Same logical schema, but repartitioning, reclustering, or reclaiming bloat on the 100M rows.<br>
_Locks and disk-doubling are the bite, not app compatibility; tooling space is totally different._<br>
`Leads to →` pg_repack, pg_squeeze, partitioning

**Recommendation:** Pin this first — `1.` and `2.` share almost no tooling overlap, and guessing wrong wastes the next 3–4 turns.

_**Next step:** Pick `1.`, `2.`, or `3.`, or describe the schema delta in one line._

---

## Notes

- Format clean across all 3 candidates. Brevity caps hold.
- Every rationale leads with the bite: _"Dual-writes and backfill become unavoidable"_ / _"Backfill is the only real risk"_ / _"Locks and disk-doubling are the bite"_.
- New `Leads to →` rule heavily exercised. `1.` uses `(+ more)` to honestly signal that the migration tooling space is broader than three names (`pgroll, pg-osc, Debezium CDC (+ more)`); `2.` and `3.` use plural destinations grouped by approach.
- Tool names accurate: `pgroll`, `pg-osc` (Shayon Mukherjee), `Debezium`, `pg_repack`, `pg_squeeze` are all real and correctly applied to their categories. No `pgroonga` hallucination.
- Branch-aware Recommendation calls out the gap explicitly: _"`1.` and `2.` share almost no tooling overlap"_ — primes the user that the tool pick is downstream of the shape pick.
- Model chose 3 candidates here vs 4 in prior passes — the additive/breaking/reorg trichotomy reads cleaner than splitting `1.` further.
