# Review: `/grill-me` vs `/cook-me` — third pass

Cook-me reruns only this time, since the only skill change since the previous review was scoped to the cook-me SKILL.md. Grill-me outputs are unchanged from the previous pass.

**The change being validated:** the brevity caps in cook-me's SKILL.md were rewritten to be explicitly scoped to two fields (brief copy and rationale) and never justify dropping structural elements like the heading, `Leads to →`, the Recommendation line, or the Next step line. The previous review's biggest cook-me miss — the state-management response dropping three structural elements in pursuit of brevity — was the failure mode this change targeted.

---

## Did the scoping fix work?

Yes, cleanly.

### State-management: the regression is fully reversed

Previous run: cook-me's state-management output omitted the `**`Candidates`**` heading, every `Leads to →` line, and the `**Recommendation:**` line. Three required elements missing.

This run: every structural element is present. Better, the candidate space itself improved — candidate `3.` is _"Server state masquerading as client state"_ with a rationale that explicitly names TanStack Query as the real answer and says the four-library question dissolves. That's the **framework-shape gap (issue C from the previous review) closed without a separate skill change** — once the agent isn't compressing structurally, it has room to surface the better framing.

The Recommendation line: _"Most React apps land in `3.` once you look closely — the 'state library' question dissolves into a server-cache question plus a small amount of UI state Context handles fine."_

This is the only prompt where grill-me beat cook-me in the previous review. It now flips.

### Migration: typography slip fixed

Previous run: every candidate was missing the `<br>` between the italic rationale and the `Leads to →` line. Pure typography slip.

This run: all three candidates have correct `<br>` placement.

### All 6 outputs are now format-clean

No `Step`/`Phase` slips. No literal `N`. No glued-progress-marker-and-blockquote. No bold-only `**1)**` markers. The "Before submitting" checklist is doing its job consistently across reruns.

### Brevity caps still hold

Spot-checks across all 6 outputs:
- Brief copy: 11–17 words (cap is ≤20)
- Rationale: 16–25 words (cap is ≤25)
- All single-sentence
- No output reached for the "extremely high bar" escape — the caps are sized correctly

State-management's candidate `3.` rationale lands right at 25 words; it's load-bearing and earns it.

### Opinion-as-content is now reliable

Three outputs carry strong, branch-aware Recommendation lines:
- **Caching:** _"If it's `4.`, I'd push back before designing anything"_
- **Monorepo:** _"`1.` and `2.` justify a monorepo; `3.` usually doesn't; `4.` is solvable either way"_
- **Background jobs:** _"If you're already reaching for orchestration or durability guarantees, say so and we'll jump to `2.`"_

The Recommendation line is no longer always a single top-pick — it's becoming an opinionated verdict-per-branch when the prompt calls for it.

---

## Per-prompt scorecard (vs unchanged grill-me)

| # | Prompt | Cook-me | Grill-me | Who wins? |
|---|---|---|---|---|
| 01 | Database | Same shape as previous, all format clean | Prose, same Postgres pick | **Cook-me** by density |
| 02 | State mgmt | Fully structured; candidate `3.` carries the TanStack-Query insight | State-type framework, same insight | **Cook-me** — flipped from previous review |
| 03 | Caching | Branch-aware Recommendation with push-back built in | One short paragraph | **Cook-me** clearly |
| 04 | Background jobs | Branch-aware Recommendation, sharp rationales | Inline workload axes | **Cook-me** clearly |
| 05 | Migration | Format fixed; one content slip (`pgroonga` is hallucinated) | Sharp "what does no downtime mean" reframe | **Grill-me** by framing, **cook-me** by structure — call it a draw |
| 06 | Monorepo | Per-branch verdicts in Recommendation | Short, sharp one-liner | **Cook-me** for the map, **grill-me** for the punch — reader's choice |

Net: cook-me ahead on 4, draw on 2, never behind. Up from 4-1-1 in the previous review.

---

## Honest issues this run

### 1. `pgroonga` hallucination on migration candidate `1.`

The Leads-to line for "Schema shape change" says `` `pgroonga`/custom triggers or logical replication ``. `pgroonga` is a full-text search extension, not a migration tool. The agent likely conflated it with `pgroll` or `pg_repack`. This is a domain-knowledge slip, not a format or skill issue — the cook-me skill can't prevent it.

### 2. Question text occasionally bloats

Database Q1: _"Before ranking databases, what's the dominant data shape..."_ — the "Before ranking databases," preamble adds nothing. The brevity cap is scoped to brief copy and rationale (correctly, per the recent change) so the Q field is unbounded. Minor; not worth changing.

### 3. Grill-me still ran terse from the previous pass

Same caveat as last review: the cook-me wins on caching, background jobs, and monorepo are partly inflated by grill-me's thinner outputs that run. A fresh grill-me rerun would close some of the gap. The state-management flip and the format fixes hold regardless.

---

## Net take

Three cook-me skill changes have now shipped in sequence (Before-submitting checklist, brevity caps with high-bar escape, scoping caps to two fields). Each addressed a specific failure mode from the prior pass. After this third pass:

- Format compliance: solid across all 6 prompts.
- Brevity caps: holding, no over-compression bleeding into structure.
- The framework-vs-candidate gap: implicitly addressed — once brevity stopped pushing the agent to shed structure, the agent had room to surface a better candidate framing without a separate rule.
- Opinion-as-content: now reliably appearing in the Recommendation line.
- Remaining gaps are content-level (a hallucinated tool name) rather than skill-level — not addressable by editing SKILL.md.

The skill iteration loop is converging. If anything is still worth tightening, it's the Q field itself: the rationale gets a cap but the question can ramble. Probably a 1-line note in the "Before submitting" checklist — but it's a small enough issue that watching it across another pass is fine before committing a change.
