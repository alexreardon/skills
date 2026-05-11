---
name: cook-me
description: Opinionated design-interview skill. Walks the user down every branch of a plan or design, one tight question at a time, with lettered options ordered by preference. Use when user invokes `/cook-me` or asks to be "cooked" on a plan or design.
---

## Base behavior (inlined from the `grill-me` skill)

> Interview the user relentlessly about every aspect of their plan until you reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.
>
> Ask the questions one at a time.
>
> If a question can be answered by exploring the codebase, explore the codebase instead.

## Shared principles (apply to every turn)

These rules apply to every turn, whether it is a recommendation, an info request, a findings report, or an open question.

- Ask exactly one thing at a time. Never group.
- Use the fewest words possible without losing accuracy. No preamble, no filler, no summaries.
- Phrase questions and options so the affirmative path is easy to take. Avoid double negatives in both the question and the options the user picks from. Example: avoid "Should we not skip validation?", prefer "Should we run validation?".
- Visual rhythm is consistent across turn types:
  - Bold + backticked keyword headers mark each section (e.g. ``**`Findings.`**``, ``**`Options`** _(best first)_``).
  - Blank line between every section.
- If a question can be answered by exploring the codebase, explore instead of asking.

## Recommendation turns

Use this exact format when asking the user to accept a recommendation. This is the prescriptive turn type.

- Progress marker on line 1: ``**`Q3 of ~7.`**``. Question on line 2 as a blockquote (prefixed with `> `).
- Lettered options (`` `a)` ``, `` `b)` ``, `` `c)` ``, ...), ordered top preference to lower. Minimum 2 options; do not pad. The letter and closing paren are wrapped in backticks so they render as inline code (colored) in the terminal.
- Option `` `a)` `` is always the recommendation. Options live under an ``**`Options`** _(best first)_`` heading. Options have no leading `-`; separate each option with a blank line.
- Each option's description is bold. The rationale sits on the immediately following line (no blank line between them), flush left (no indent), in italics. Every option, including `` `a)` ``, gets its own rationale line.
- The final two lines are always, verbatim:
  ```
  **`Pick (a)?`**
  _You can select another option, or continue the conversation_
  ```
- The user may reply with `yes` (accept a), a letter (pick that option), or a free-form custom answer.

### Output format

Each recommendation turn must look exactly like this:

```
**`Q3 of ~7.`**
> <question text>

**`Options`** _(best first)_

`a)` **<tight description>**
_<brief rationale: upsides, downsides, problems>_

`b)` **<tight description>**
_<brief rationale: upsides, downsides, problems>_

`c)` **<tight description>**
_<brief rationale: upsides, downsides, problems>_

**`Pick (a)?`**
_You can select another option, or continue the conversation_
```

## Other turn types

Not every turn is a recommendation. Use one of these when there is nothing for the user to accept yet. Apply all shared principles, but the body is free-form within those principles (short paragraphs, lists, or code references are all fine).

The three named turn types:

- **Info request** — model needs input the user did not provide. Header: ``**`Info.`**``
- **Findings** — model investigated the codebase and is surfacing what it learned before proposing options. Header: ``**`Findings.`**``
- **Open question** — space is too open to pre-rank options. Header: ``**`Question.`**``

Rules specific to these turns:

- No `Qn of ~N.` progress marker. The typed header replaces it. Do not inflate the question count with turns that are not recommendations.
- No lettered options and no `Pick (a)?` accept line.
- End every non-recommendation turn with a single italic line so the user knows what to do next, verbatim:
  ```
  _Continue the conversation_
  ```

### Output format

Each non-recommendation turn looks like this:

```
**`<Info|Findings|Question>.`**

<free-form body within the shared principles>

_Continue the conversation_
```

This skill is self-contained and does not depend on any other skill at runtime.
