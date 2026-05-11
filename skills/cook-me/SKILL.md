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
- Visual rhythm is consistent across turn types:
  - Bold + backticked keyword markers anchor structural sections (e.g. ``**`Q3 of ~7.`**``, ``**`Options`** _(↓ most to least recommended)_``).
  - Bold prose markers (no backticks) introduce the model's voice (e.g. `**Recommendation:**`, `**Assumption:**`) — matches grill-me's authoring style.
  - The final action line is italic with a bold `**Next step:**` prefix — only the label is bold; the action phrase that follows is italic only (e.g. ``_**Next step:** Accept `1.`, pick another, or correct the assumption._``). Makes the required action visually unmistakable so the user never has to guess what to do.
  - Inline references to options or candidates in prose are backticked (e.g. _"start with `1.` and add `2.` later"_) — keeps them visually distinct and prevents accidental list rendering.
  - Blank line between every section.
- If a question can be answered by exploring the codebase, explore instead of asking.

## Recommendation turns

Use this exact format when asking the user to accept a recommendation.

- Progress marker on line 1: ``**`Q3 of ~7.`**``. Question on line 2 as a blockquote (prefixed with `> `).
- Numbered options (`` `1.` ``, `` `2.` ``, `` `3.` ``, ...), ordered top preference to lower. Use as many options as the decision warrants — one is fine if there's truly one path. Do not pad. The number and closing paren are wrapped in backticks so they render as inline code (colored) in the terminal.
- Option `` `1.` `` is always the recommendation. Options live under an ``**`Options`** _(↓ most to least recommended)_`` heading. Options have no leading `-`; separate each option with a blank line.
- Each option spans three lines: a bold name, the brief copy (plain) on the next line describing what the option is / how it works, then the rationale in italics on the third line. Typography (bold / plain / italic) is what distinguishes the lines — no extra markers, prefixes, or indentation. No blank lines between the three; blank line between options.
- **Brevity caps (scoped to two fields):** brief copy ≤20 words, rationale ≤25 words, one sentence each. These limits apply ONLY to those two fields — they never justify dropping a required structural element (the `**`Options`**` heading, the `**Recommendation:**` line, the `_**Next step:**_` line, or any other format-spec element). Bust a cap only when content is genuinely uncompressible — extremely high bar, not a routine escape.
- The rationale names the most non-obvious implication of picking this option — surprising operational consequences, failure modes, hidden costs — not generic directional language ("pushes toward X, hard rejects").
- Optionally, one or more `**Assumption:**` lines (bold prose, no backticks) appear first — each names a load-bearing assumption the recommendation rides on plus the model's expected value (e.g. _"`**Assumption:**` Your traffic has legitimate bursts (most APIs do)."_). Stack lines when the recommendation rides on multiple guessable defaults that each independently could change the pick. Soft limit: if you're tempted to write a third stacked line, that's a signal intent isn't pinned and you should defer to an exploration turn first. Include only when the recommendation changes if the assumption is wrong; skip when the pick is robust to any reasonable user situation.
- The recommendation line leads with `**Recommendation:**` (bold prose, no backticks — same marker as exploration turns) followed inline by the brief recommendation, referencing the top option `` `1.` `` directly or summarizing in one line. When an `**Assumption:**` is present, the recommendation reads as a consequence of it.
- The final line is a contextual italic action line with a bold `**Next step:**` prefix; the action phrase after the colon is italic only (e.g. ``_**Next step:** Accept `1.`, pick another, or correct the assumption._``). Makes the required action unmistakable.
- The user may reply with `yes` (accept the recommendation), a number (pick that option), or a free-form custom answer (e.g. correcting the assumption).

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

Use this format to surface intent, candidate space, codebase findings, or any question that isn't yet a ranked decision. Exploration turns share recommendation turns' authoring discipline (surface candidates, expose model thinking, state a recommendation) — the difference is the body shape and how the user is invited to respond.

- Progress marker on line 1: ``**`Q3 of ~7.`**`` — same counter as recommendation turns. Every question turn contributes to the same `Q of ~N` count.
- Question on line 2 as a blockquote (prefixed with `> `).
- Candidates live under a ``**`Candidates`** _(↓ most to least promising)_`` heading — visually parallel to recommendation turns' ``**`Options`** _(↓ most to least recommended)_`` heading, but the criterion is softened to "promising" to honor that these are surfaced framings (informational, the model's current thinking) rather than ranked picks.
- Candidates use numbered markers `` `1.` ``, `` `2.` ``, `` `3.` ``, ... in backticks. Same marker shape as recommendation turns — but exploration candidates are informational, not a ranked accept list. Each candidate spans three lines: a bold label, the brief content (plain), then the rationale in italics. Typography (bold / plain / italic) is what distinguishes the lines — no extra markers, prefixes, or indentation. No blank lines between the three; blank line between candidates.
- **Brevity caps (scoped to two fields):** brief content ≤20 words, rationale ≤25 words, one sentence each. These limits apply ONLY to those two fields — they never justify dropping a required structural element (the `**`Candidates`**` heading, the `` `Leads to →` `` line where the destination is clear, the `**Recommendation:**` line, the `_**Next step:**_` line, or any other format-spec element). Bust a cap only when content is genuinely uncompressible — extremely high bar, not a routine escape.
- The rationale names the most non-obvious implication of this framing — surprising consequences, failure modes, hidden costs, or what it forces downstream — not generic directional language.
- Optionally, each candidate gets a 4th line naming where it leads — the downstream pick or the next decision this candidate would force. Format: `` `Leads to →` <destination> `` (backticked label including the arrow, then plain destination, optionally followed by a brief parenthetical why). Include per-candidate when the destination is clear and short (a tool name, a strategy name, occasionally a brief parenthetical); this gives the reader signal to short-circuit (commit to the downstream answer directly) or sanity-check (see where their upstream pick is heading) without waiting for the next turn. Omit on candidates where the downstream pick isn't yet clear, where it has the same shape across all candidates, or where the destination needs more than a tail-of-line. Inlining the leads-to into each candidate (rather than collecting them in a separate block) keeps the reader's eye local — no mental mapping between the candidate at the top of the section and a matching `N)` reference further down.
- Penultimate line leads with `**Recommendation:**` (bold prose marker, not backticked — mirrors grill-me's "My recommendation:" authoring style), followed inline by the brief recommendation itself (the analog of recommendation turns' `` `1.` `` pick). One line is fine; longer is fine when the recommendation spans multiple candidates.
- Final line is a contextual italic action line with a bold `**Next step:**` prefix; the action phrase after the colon is italic only (e.g. ``_**Next step:** Tell me the primary driver, or steer free-form._``). Makes the required action unmistakable. Single line.
- The user may reply with `yes` (accept the recommendation), a free-form answer, or steer.

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

Re-read the output against this checklist. The same slips recur across runs — catching them once before send is cheap; the cost of an inconsistent turn compounds across the session.

- **Progress marker** is ``**`Qn of ~N.`**`` — `Q` only (never `Step`, `Phase`, `Part`), and a real integer (never the literal letter `N`).
- **Progress marker and question are on separate lines.** The marker is NOT in a blockquote; the question IS. Do not write `> **Q1.** What...?` on one line.
- **Every candidate / option starts with a backticked numbered marker** — `` `1.` ``, `` `2.` ``, etc. Not `**1)**` (bold-only), not `1.` (plain).
- **Italic uses single underscores** `_text_`, not `_*text*_` or `*_text_*` (those are bold-italic). The rationale line is italic-only.
- **Each candidate / option uses `<br>` after the bold label/name line and the plain content/copy line** so the 3-line (or 4-line, with `` `Leads to →` ``) shape preserves when rendered as markdown.
- **The rationale (italic line) names a SURPRISING / non-obvious implication** — operational gotcha, failure mode, hidden cost — not generic directional language like "pushes toward X, hard rejects."
- **Brevity caps (scoped): brief copy/content ≤20 words, rationale ≤25 words, one sentence each.** Count words on every option/candidate. The caps apply ONLY to those two fields — they never justify dropping the heading, `` `Leads to →` `` line (where the destination is clear), `**Recommendation:**` line, or `_**Next step:**_` line. Bust only if content is genuinely uncompressible (extremely high bar).
- **The final line starts with `_**Next step:**`** — italic-wrapping, bold label, italic action phrase after the colon.
