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
> What's forcing the migration — new columns, type changes, normalization, partitioning, or something else?

**`Candidates`** _(↓ most to least promising)_

`1.` **Additive schema change**<br>
New columns or indexes; old readers stay valid throughout.<br>
_Backfill is the only real risk; you can ship app changes lazily since old code keeps working._<br>
`Leads to →` expand-contract with a backfill job

`2.` **Type or constraint change**<br>
Column retypes, NOT NULL adds, FK additions on existing data.<br>
_Validation cost dominates; a single bad row blocks the whole cutover unless you stage validation separately._<br>
`Leads to →` shadow column + dual-write

`3.` **Restructure or split**<br>
Normalization, table split, or partitioning of the existing table.<br>
_Read paths fan out; every query and join touching the table needs an audit, not just writers._<br>
`Leads to →` shadow table + logical replication or triggers

`4.` **Full rewrite**<br>
New table with materially different shape and semantics.<br>
_Dual-write window is long and app-visible; rollback means keeping both tables hot for weeks._<br>
`Leads to →` CDC-style sync (e.g. pg_logical, Debezium)

**Recommendation:** Pin the shape of the change before picking a strategy — "no downtime on 100M rows" looks very different for `1.` vs `4.`, and the wrong tool here wastes weeks.

_**Next step:** Pick a candidate or describe the actual schema delta._

---

## Notes

- Format clean across all 4 candidates. Brevity caps hold (candidates 9–13 words, rationales 14–18 words).
- Every rationale leads with the bite: _"Backfill is the only real risk"_ / _"Validation cost dominates"_ / _"Read paths fan out"_ / _"Dual-write window is long and app-visible"_. None reach for directional verbs.
- Tool names are accurate: `pg_logical`, `Debezium`, expand-contract are all real and correctly applied. The `pgroonga` hallucination from earlier passes does not return.
- Branch-aware Recommendation calls out the gap explicitly: _"`1.` vs `4.` looks very different"_ — primes the user that the tool pick is downstream of the shape pick.
