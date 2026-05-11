# Comparisons: `/grill-me` vs `/cook-me`

Sub-agent runs comparing the two skills across 6 prompts of varying shape. Each file contains the verbatim first-turn output from both skills plus brief notes.

## The prompts

1. [Database choice](./01-database.md) — pre-enumerated options (Postgres/Mongo/Dynamo), common scenario
2. [State management](./02-state-management.md) — pre-enumerated options (Redux/Zustand/Jotai/Context), app-shape-dependent
3. [Caching strategy](./03-caching.md) — open exploration, multi-axis (where + how)
4. [Background jobs](./04-background-jobs.md) — open exploration, tooling-heavy
5. [Migration strategy](./05-migration.md) — specific technical scenario, multi-step
6. [Monorepo decision](./06-monorepo.md) — strategic yes/no, multi-driver

See [REVIEW.md](./REVIEW.md) for the full assessment and the pass-by-pass iteration history (passes 5–9) that converged on the current state.

---

Cook-me outputs in each comparison file reflect the latest validated run (pass 9 — all 6 prompts format-clean, every rationale leads with the bite, plural `Leads to →` destinations exercised throughout). Grill-me outputs are unchanged from the original capture.

All outputs captured via parallel sub-agent runs (`general-purpose` agent, simulating Claude Code) against the latest [cook-me SKILL.md](../skills/cook-me/SKILL.md) and the upstream [grill-me SKILL.md](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md).
