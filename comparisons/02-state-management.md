# 02. React state management library

**Prompt:**
> "I'm building a React app and trying to choose a state management library. Should I use Redux Toolkit, Zustand, Jotai, or React Context?"

---

## `/grill-me` output

Before picking a library, the most load-bearing question is about the shape of your state itself, because that determines which tool actually fits.

**What kind of state dominates your app?**

a) Mostly **server state** (data fetched from APIs, cached, refetched) with a small amount of local UI state
b) Mostly **client state** (complex local logic, derived values, cross-component coordination) — e.g. a builder, editor, or dashboard with rich interactions
c) Mostly **simple shared UI state** (theme, auth user, modal open/closed, a few global flags)
d) A genuinely large, deeply nested **normalized domain model** (entities with relationships, lots of updates across many slices)

My recommendation: **(a)** — for the vast majority of React apps today, what people call "state management" is actually server state, and the right answer is to pair a server-state library (React Query / TanStack Query or RTK Query) with a lightweight client store only where needed. Reaching for Redux Toolkit / Zustand / Jotai before answering this question is the most common mistake I see, because each of those libraries shines in a different one of these four scenarios.

Which one is yours?

---

## `/cook-me` output

**`Q1 of ~6.`**
> What kind of app are you building, and what's pulling you toward a dedicated library?

**`Candidates`** _(↓ most to least promising)_

`1.` **Server state dominates**<br>
Most of your state is API data — lists, details, mutations, caching.<br>
_The library war is moot here; Redux/Zustand will reinvent caching badly compared to a server-state tool._<br>
`Leads to →` React Query, SWR, RTK Query

`2.` **Cross-cutting client state**<br>
Auth, theme, modals, wizards — modest shared state across unrelated trees.<br>
_Context handles this fine until re-renders bite; the bite arrives later than people expect._<br>
`Leads to →` Context, Zustand

`3.` **Complex client-only state**<br>
Editors, builders, multi-step flows with derived/atomic state and undo.<br>
_Granular reactivity becomes load-bearing; coarse stores cause cascading re-renders that profile poorly._<br>
`Leads to →` Jotai, Zustand, Redux Toolkit

`4.` **Team/scale-driven**<br>
Large team, strict conventions, devtools and middleware matter more than ergonomics.<br>
_Boilerplate stops being the cost; debuggability and predictable patterns dominate the calculus._<br>
`Leads to →` Redux Toolkit

**Recommendation:** Most React apps land in `1.` — the four libraries you listed solve a different problem than the one you probably have. Confirm the shape before we rank.

_**Next step:** Pick a candidate, or describe the state that's actually painful._

---

## Notes

- Format clean across all 4 candidates. Brevity caps hold.
- Every rationale leads with the bite: _"The library war is moot here"_ / _"Context handles this fine until re-renders bite"_ / _"Granular reactivity becomes load-bearing"_ / _"Boilerplate stops being the cost"_.
- New `Leads to →` rule heavily exercised — 3 of 4 candidates use plural destinations (`React Query, SWR, RTK Query`, `Context, Zustand`, `Jotai, Zustand, Redux Toolkit`). The user can short-circuit by recognizing any name on the list.
- Server-vs-client framing stays as `1.` and the Recommendation (consistent placement across all passes).
