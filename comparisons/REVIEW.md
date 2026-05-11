# Review: `/grill-me` vs `/cook-me` — passes 5–8 (consolidation, regressions, fixes)

This review covers the iteration loop that ran after the consolidation pass to cook-me's SKILL.md (passes 1–4 are summarized in the file's history; this document is the live record). Grill-me outputs unchanged from earlier passes.

**The story in one paragraph:** Pass 4 was clean (5/6 wins, 1 draw, 0 losses) right after the consolidation that lifted shared rules into a `Turn format` section and slimmed the `Before submitting` checklist from 8 items to 3. Pass 5 surfaced two regressions: a structural `<br>` slip on 2/6 outputs (the slim checklist no longer caught it) and a content drift where caching's rationales used directional language (_"pushes toward X"_) that an earlier checklist item used to flag. Three iterations of SKILL.md edits and three more passes were needed to close both — the rationale fix in particular required moving from a verb-list ban (which the model dodged with synonyms) to a shape-based rule (which the model dodged with creative paraphrasing) to a worked WRONG → RIGHT example (which finally landed).

---

## Pass-by-pass summary

| Pass | What changed in SKILL.md | Format-clean | Content-clean | Net |
|---|---|---|---|---|
| 4 | Consolidation: shared `Turn format` section, slim 3-item `Before submitting` checklist | 6/6 | 5/6 (mild caching drift) | Clean baseline |
| 5 | (no change) | 4/6 — `<br>` regression on background jobs and migration | 5/6 — directional language back on caching, this time on `1.` | Two regressions surface |
| 6 | Added 2 items to checklist: `<br>` placement, rationale anti-pattern (verb list) | 6/6 — `<br>` regression fixed | 5/6 — caching `2.` dodged the verb list with _"Shifts the conversation"_ | One down, one persisting |
| 7 | Reworded anti-pattern rule from verb-list to shape-based ("first clause must be the consequence") + expanded verb list | 6/6 | 5/6 — caching dodged shape rule with _"Pulls cache close to..."_ + _"Pushes state to..."_ | Same caching drift, different verbs |
| 8 | Added worked WRONG → RIGHT example to the rule, showing how to reorder clauses so the bite leads | **6/6** | **6/6** | **Clean — loop closes** |

Pass 8 is the canonical state captured in the comparison files. Each comparison file's `## Notes` reflects pass-8 outputs.

---

## What the iteration revealed about the model

### 1. Verb-list bans don't survive contact with creative paraphrasing

The pass-6 checklist item explicitly listed _"pushes you toward X"_, _"points you at Y"_, _"leads you to Z"_, _"forces a Q"_. The pass-7 caching output reached for _"pulls cache close to..."_, _"shifts the conversation to..."_, _"drives you toward..."_. None of these were on the list, but all are structurally identical — describing the option's architectural fingerprint instead of what bites.

Lesson: when a rule lists specific bad phrasings, the model treats the list as exhaustive. It will keep producing structurally identical bad output with novel verbs. The rule must describe the *shape*, not enumerate examples.

### 2. Even shape-based rules need worked examples for the hard cases

The pass-7 rule was already shape-based — _"first clause must be the consequence, not where the option lands architecturally."_ It worked on every prompt except caching. Caching is structurally about WHERE to put the cache, so the model's first instinct keeps reaching for location-based rationales.

The fix that landed: a worked WRONG → RIGHT rewrite using caching-flavored examples, showing that the same information can be reordered so the bite leads. The location is allowed to appear, just not as the lead.

```
❌ "Pushes state to client/edge; invalidation now spans devices you don't control."
✅ "Invalidation now spans devices you don't control; client/edge is the only place state can live."
```

Pass 8 caching: _"Staleness becomes the dominant tradeoff; reads are easy to cache but invalidation is where you'll bleed."_ — exactly the shape the worked example trained for.

### 3. Slimming the checklist costs more than it saves

The original 8-item checklist was deliberately redundant with the body rules — that redundancy was its job. Slimming to 3 items in the consolidation pass (passes 1–4) read like a clean simplification but left two failure modes uncovered: `<br>` placement and rationale-content shape. Both came back as regressions in pass 5.

The current checklist is 5 items. Each one survived an iteration cycle by catching a real recurring slip:

- **Progress marker shape** — caught in original 8-item checklist; never regressed.
- **Italic single underscores** — caught in original; never regressed.
- **`<br>` placement** — was in original 8-item; dropped in slim 3-item; regression in pass 5; added back in pass 6.
- **Rationale FIRST CLAUSE shape (with worked example)** — was a parenthetical in original; dropped in slim 3-item; regression on caching across passes 4–7; rewritten three times across passes 6–8 before landing.
- **Brevity caps** — caught in original; never regressed.

Net length: 5 items vs the original 8. We kept the duplication-reduction win on the checklist (3 items dropped) without the regressions that the most aggressive trim caused.

---

## Per-prompt scorecard (vs unchanged grill-me) — pass 8 final state

| # | Prompt | Cook-me | Grill-me | Who wins? |
|---|---|---|---|---|
| 01 | Database | All rationales lead with the bite, branch-aware Recommendation | Prose, same Postgres pick | **Cook-me** by density |
| 02 | State mgmt | Server-data framing as `1.`, _"client lib choice barely matters"_ as the lead rationale | State-type framework, same insight | **Cook-me** by structure |
| 03 | Caching | All four rationales lead with the bite (the prompt that drove the iteration loop) | One short paragraph | **Cook-me** clearly |
| 04 | Background jobs | Bite-led rationales, branch-aware Recommendation | Inline workload axes | **Cook-me** by structure |
| 05 | Migration | Sharp candidate set, accurate tool names (`pg_logical`, `Debezium`), branch-aware verdict | Sharp "what does no downtime mean" reframe | **Draw** — different question shapes, both valid |
| 06 | Monorepo | All five candidates name cheaper non-monorepo fixes per rationale; explicit verdict ranking | Short, sharp one-liner | **Cook-me** for the map, **grill-me** for the punch — reader's choice |

Net: cook-me ahead on 4, draws on 2, never behind. Same shape as the pre-consolidation pass-3 review (4-2-0); the iteration loop restored the wins that pass-5/6/7 partial regressions had eroded.

---

## Honest issues this run

### 1. Caching took 4 iterations to land

Passes 5, 6, 7 all had directional-language drift on caching. The drift only fully cleared in pass 8 after the worked-example fix. This is a real signal that the rule was under-specified for prompts where the candidate space is fundamentally about location. Specific-domain examples in the checklist may be needed for other location-shaped prompts (queues, services, infrastructure choices) — worth watching whether those prompts surface the same drift in future runs.

### 2. Variance in candidate count and `~N` value

Across passes, the same prompts have ranged from 3 to 5 candidates and from `~6` to `~8` for the question count. The spec allows this (_"use as many as the decision warrants"_, _"real integer for N"_) but the comparison files now show different cuts of the same space across passes. This is a feature, not a bug — different runs surface different framings. But if reproducibility-of-framing matters for any consumer, the spec doesn't pin it.

### 3. Grill-me caveat unchanged

Grill-me outputs are still from the original pass and run thinner on caching, background jobs, monorepo. A fresh grill-me rerun would close some of the cook-me wins. The format-compliance and rule-rigor findings hold regardless.

---

## Net take on the consolidation + iteration loop

The duplication reduction made cook-me's SKILL.md ~17% shorter (131 → ~109 lines after passes 1–4) but introduced two regressions because the slim 3-item checklist was missing two slips that recur. The fix wasn't to undo the consolidation — the `Turn format` section and the smaller variant sections still earn their place. The fix was to restore checklist items that catch specific repeating failure modes, and to discover (through caching's stubbornness) that one of those items needs a worked example to actually land.

Final SKILL.md state:

- 1 `Turn format` shared section (no duplication with variant sections)
- 2 variant sections (`Recommendation turns`, `Exploration turns`) — each ~4 bullets covering only deltas
- 5-item `Before submitting` checklist (was 3 after pass 4 consolidation, now 5 after passes 6+8 added back what was needed)
- Both code blocks intact

The loop converged. If anything is still worth tightening, it's whether the worked-example pattern from the rationale rule should also be applied to other recurring slips (e.g. show a WRONG `Step 3 of ~5` vs RIGHT `Q3 of ~5` example for the progress marker rule). That's a future-pass question; current state is clean.
