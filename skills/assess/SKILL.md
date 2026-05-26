---
name: assess
description: Review work on the current branch or in the current context. Use when the user invokes "/assess" or asks to assess their changes.
---

# Assess

## When to Use This Skill

- User runs `/assess`
- User asks to assess or review the current changes or work

## How to communicate with me

Use as few words as possible without losing meaning. Sacrifice grammar if needed. Use dot pots to make things easier to read. Use emojis to help understanding to.

## Steps

### Gain understanding

Do the following steps without messaging me:

1. Figure out the changes to be assessed. If no explicit scope is provided, look at all staged and unstaged changes on `git` (`git diff` for unstaged changed and `git diff --staged` for staged changes)
2. Run `git log` to understand recent git context (eg `git log -3 -p`)
3. Pull the users personal `AGENT.md` and `CLAUDE.md` and the project `AGENT.md` and `CLAUDE.md` into context
4. If there are any links in the `AGENT.md` and `CLAUDE.md` files, pull any linked files into context that are likely to be relevant to the files being assessed
5. Try to understand what has changed and why.
6. If you have any meaningful questions about what changed or why, ask me one question at a time. Continue doing this until you have a high level of confidence that you understand the intent of the change.

One done, make this message:

```md
## What changed

- change 1 (short description)
- change 2 (short description)
```

### Review changes

As a result of your review, I want you to make a plan. However, I want to work with you on creating the plan.

I want you to look for the following:

- Any violations of `AGENT.md` and `CLAUDE.md` practices
- Bugs or logic errors
- Missing error handling
- Security issues
- Consistency with surrounding code
- Anything that looks unintentional (debug code, leftover TODOs, accidental deletions)
Opportunities to simplify or clean up the code (dead code, unnecessary complexity, unclear naming)
- Opportunities to consolidate abstractions (duplicated logic, similar helpers that could be unified, patterns that should be shared)
- Gaps in testing

For anything that clearly should be done, add it to the plan. Then make the following message:

```md
## Changes that should be done

- update 1 _(tiny description)_
- update 2 _(tiny description)_
```

If you have any questions about whether something should be done, ask me questions one at a time whether to add them to the plan or not. Make it clear which path you recommend - add "Recommended" to the option. List out options in the order of your recommendation.

When you are done, ask: "are you happy with these choices"?

- default: "yes" (continue) (no input will be considered a "yes")
- alternative: "no" (ask questions again)

Once done with this loop, list out all changes:

```md
## Changes that will be done

- change 1 [ai] _(tiny description)_
- change 2 [user] _(tiny description)_

## Changes that won't be done

- change 3 [user] _(tiny description)_
```

## Plan

Plan should include

- What changed
- Changes that will be done
- Changes that won't be done
- Verification steps (how we know if the updates worked). Ensure you run the minimum of things needed