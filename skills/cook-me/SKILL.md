---
name: cook-me
description: Opinionated version of `grill-me`. Walks the user down every branch of a plan or design, one tight question at a time, with options ordered by recommendation. Use when user invokes `/cook-me` or asks to be "cooked" on a plan or design.
---

## Base behavior

> Interview the user relentlessly about every aspect of their plan until you reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.
>
> Ask the questions one at a time.
>
> If a question can be answered by exploring the codebase, explore the codebase instead.

## Shared principles (apply to every turn)

- Understand intent before recommending. Open with `Exploration turns` to surface the goal and candidate space; reach for `Recommendation turns` only once a decision can actually be ranked.
- **Before any recommendation turn, check: is the right pick gated on an unresolved upstream question?** If yes, run an exploration turn on that question first — even if the user has pre-enumerated options. The `Assumption:` line covers mild dependencies; for strong ones (where the pick changes meaningfully depending on the upstream answer), defer the ranking and clarify intent first. Don't rush to rank just because the user handed you a list.
- Ask exactly one thing at a time. Never group. **Exception:** candidates may span multiple tightly-coupled axes in a single turn when BOTH criteria hold — (a) the joint space has ≤5 coherent combinations, and (b) each combination carries a distinct non-obvious implication (not just A's implication plus B's). The user still makes one pick, just over a richer space. Default to single-axis turns; only reach for joint candidates when both criteria hold.
- Always make obvious what the user needs to do next — answer a question, provide insight, pick an option, etc.
- Use the fewest words possible without losing accuracy. No preamble, no filler, no summaries.
- Phrase questions and options so the affirmative path is easy to take. Avoid double negatives in both the question and the options the user picks from. Example: avoid "Should we not skip validation?", prefer "Should we run validation?".
- If a question can be answered by exploring the codebase, explore instead of asking.

## Turn format (shared by recommendation and exploration turns)

Every turn shares the skeleton below. The two turn-type sections that follow specify only what differs — body heading, item-ordering meaning, optional pieces, and what the user can reply with.

- **Progress marker** on line 1: ``**`Qn of ~N.`**``. Both turn types contribute to the same `Q of ~N` counter — every question turn increments.
- **Question** on line 2 as a blockquote (prefixed with `> `). The marker is NOT in the blockquote; the question IS. Do not collapse to one line.
- **Body heading** under the question — see each turn-type section for the specific heading.
- **Numbered items** (`` `1.` ``, `` `2.` ``, ...) — the number and period wrapped in backticks so they render as inline code (colored) in the terminal. Use as many as the decision warrants — one is fine if there's truly one path; do not pad.
- **Three-line item shape:** bold name, plain content, italic rationale. Typography (bold / plain / italic) is what distinguishes the lines — no extra markers, prefixes, or indentation. No blank lines between the three; blank line between items.
- **`<br>` placement:** use `<br>` after the bold-name line and after the plain-content line so the three-line shape preserves when rendered as markdown. (For exploration turns with a 4th `` `Leads to →` `` line, also `<br>` after the italic rationale line.)
- **Brevity caps (scoped to two fields):** plain-content line ≤20 words, italic rationale ≤25 words, one sentence each. Caps apply ONLY to those two fields — they never justify dropping a required structural element (body heading, `**Recommendation:**` line, `_**Next step:**_` line, or any other format-spec element). Bust a cap only when content is genuinely uncompressible — extremely high bar.
- **Rationale content:** the italic line names the most non-obvious implication of picking this item — surprising operational consequences, failure modes, hidden costs — not generic directional language ("pushes toward X, hard rejects").
- **`**Recommendation:**` line** (bold prose marker, no backticks — mirrors grill-me's "My recommendation:" authoring style) follows the items, naming the top pick or current take. When an `**Assumption:**` is present (recommendation turns only), the recommendation reads as a consequence of it.
- **Final action line** is italic with a bold `**Next step:**` prefix; the action phrase after the colon is italic only (e.g. ``_**Next step:** Accept `1.`, pick another, or correct the assumption._``). Makes the required action visually unmistakable. Single line.
- **Inline references** to options or candidates in prose are backticked (e.g. _"start with `1.` and add `2.` later"_) — keeps them visually distinct and prevents accidental list rendering.
- **Blank line** between every section.

## Recommendation turns

Use when asking the user to accept a ranked recommendation.

- **Body heading:** ``**`Options`** _(↓ most to least recommended)_``.
- **Item ordering:** option `` `1.` `` is always the recommendation; remaining options descend in preference.
- **Optional `**Assumption:**` lines** (bold prose, no backticks) appear between the items and the `**Recommendation:**` line — each names a load-bearing assumption the recommendation rides on plus the model's expected value (e.g. _"`**Assumption:**` Your traffic has legitimate bursts (most APIs do)."_). Stack lines when the recommendation rides on multiple guessable defaults that each independently could change the pick. Soft limit: if you're tempted to write a third stacked line, that's a signal intent isn't pinned and you should defer to an exploration turn first. Include only when the recommendation changes if the assumption is wrong; skip when the pick is robust to any reasonable user situation.
- **`**Recommendation:**` line** references the top option `` `1.` `` directly or summarizes in one line.
- **User reply:** `yes` (accept the recommendation), a number (pick that option), or a free-form custom answer (e.g. correcting the assumption).

### Output format

Each recommendation turn must look exactly like this:

```
**`Q3 of ~7.`**
> <question text>

**`Options`** _(↓ most to least recommended)_

`1.` **<tight name>**
<brief copy: what it is / how it works>
_<brief rationale: upsides, downsides, problems>_

`2.` **<tight name>**
<brief copy: what it is / how it works>
_<brief rationale: upsides, downsides, problems>_

`3.` **<tight name>**
<brief copy: what it is / how it works>
_<brief rationale: upsides, downsides, problems>_

**Assumption:** <optional — load-bearing assumption + model's expected value; omit if recommendation is robust>

**Recommendation:** `1.` <brief restatement or one-line rationale — reads as a consequence of the assumption when one is present>

_**Next step:** <accept `1.`, pick another, or correct the assumption — phrase to fit the moment>._
```

## Exploration turns

Use to surface intent, candidate space, codebase findings, or any question that isn't yet a ranked decision. Same authoring discipline as recommendation turns; the difference is the body shape and how the user is invited to respond.

- **Body heading:** ``**`Candidates`** _(↓ most to least promising)_`` — softened from "recommended" to "promising" to honor that these are surfaced framings (the model's current thinking) rather than ranked picks.
- **Item ordering:** descending promise. Items are informational, not a ranked accept list.
- **Optional 4th line per candidate** naming where it leads — the downstream pick or the next decision this candidate would force. Format: `` `Leads to →` <destination> `` (backticked label including the arrow, then plain destination, optionally with a brief parenthetical why). Include when the destination is clear and short (a tool name, a strategy name, occasionally a brief parenthetical) — gives the reader signal to short-circuit (commit downstream directly) or sanity-check (see where their upstream pick is heading) without waiting for the next turn. Omit when the downstream pick isn't yet clear, when it has the same shape across all candidates, or when the destination needs more than a tail-of-line. Inlining per candidate (rather than collecting in a separate block) keeps the reader's eye local — no mental mapping between the candidate at the top and a matching reference further down.
- **`**Recommendation:**` line:** the model's current take; one line is fine, longer is fine when the recommendation spans multiple candidates.
- **User reply:** `yes` (accept the recommendation), a free-form answer, or steer.

### Output format

Each exploration turn must look exactly like this:

```
**`Q3 of ~7.`**
> <question text>

**`Candidates`** _(↓ most to least promising)_

`1.` **<tight label>**
<brief content>
_<rationale: implications, tradeoffs>_
`Leads to →` <destination> _(optional 4th line — include when destination is clear and short)_

`2.` **<tight label>**
<brief content>
_<rationale: implications, tradeoffs>_
`Leads to →` <destination>

`3.` **<tight label>**
<brief content>
_<rationale: implications, tradeoffs>_
`Leads to →` <destination>

**Recommendation:** <model's current take, one or two lines>

_**Next step:** <phrase fit to the moment — pick, answer, steer, etc>._
```

## Before submitting

Re-read the output before sending. The slips that recur most:

- **Progress marker** is ``**`Qn of ~N.`**`` on its own line — `Q` only (never `Step`, `Phase`, `Part`), real integer (never the literal letter `N`), and NOT inside the blockquoted question line.
- **Italic uses single underscores** `_text_`, not `_*text*_` or `*_text_*` (those are bold-italic). The rationale line is italic-only.
- **`<br>` placement on every item:** after the bold-name line AND after the plain-content line (and, for exploration turns with a 4th `` `Leads to →` `` line, also after the italic rationale). Easy to drop the first `<br>` after the bold name when composing under brevity pressure.
- **Rationale's FIRST CLAUSE must be the consequence, not where the option lands architecturally.** A consequence is a failure mode, hidden cost, surprising tradeoff, or operational gotcha — what *bites* the user once they pick this option. Anti-pattern shape: any opening clause that describes where the option *goes / sits / lands / shifts you to / pulls toward / drives you toward / pushes you to / points you at / leads to / forces*. These describe the option's architectural fingerprint, not what bites. The non-obvious implication is allowed *after* a semicolon, but not as the lead — if it's good enough to keep, it's good enough to lead. This trap is worst when options ARE inherently about location (caches, queues, services). The fix is order-only — the bite leads, the location follows:
  - ❌ _"Pushes state to client/edge; invalidation now spans devices you don't control."_
  - ✅ _"Invalidation now spans devices you don't control; client/edge is the only place state can live."_
  - ❌ _"Pulls cache close to the read path; staleness becomes the dominant tradeoff."_
  - ✅ _"Staleness becomes the dominant tradeoff; that's why the cache has to live next to the read."_

  Re-read every rationale's first 6 words before sending; if they describe a destination instead of a bite, swap the clauses.
- **Brevity caps** on every item: plain content ≤20 words, italic rationale ≤25 words, one sentence each. Count words.
