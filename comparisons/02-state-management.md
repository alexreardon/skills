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
> What does this app actually need state to do?

**`Candidates`** _(↓ most to least promising)_

`1.` **Server data dominates**<br>
Most "state" is fetched from APIs — lists, details, mutations, caching, refetching.<br>
_Client lib choice barely matters once a server-cache owns the hot path; picking Redux/Zustand here solves the wrong problem._<br>
`Leads to →` React Query / RTK Query (then a tiny client store for the rest)

`2.` **Cross-tree client state**<br>
Auth, theme, modals, wizards, selections shared across distant components.<br>
_Prop-drilling pain is real but small; the pick hinges on whether updates are frequent or rare._<br>
`Leads to →` Zustand or Jotai (frequent updates) vs Context (rare updates)

`3.` **Complex local domain logic**<br>
Undo/redo, collaborative editing, derived graphs, normalized entities with relationships.<br>
_Reducers and middleware earn their weight here; lighter libs force you to rebuild the same machinery yourself._<br>
`Leads to →` Redux Toolkit

`4.` **Mostly local component state**<br>
Forms, toggles, hover — state lives near where it's used.<br>
_A library is overhead you'll regret; `useState` plus a sprinkle of Context covers it._<br>
`Leads to →` React built-ins only

**Recommendation:** Most React apps land on `1.` — the question "which client store" usually evaporates once a server-cache library is in place. Worth confirming before ranking client libs.

_**Next step:** Pick the candidate that fits, or describe the app in a sentence and I'll place it._

---

## Notes

- Server-vs-client framing stays as `1.` and the recommendation (consistent placement across passes 4–8).
- Every rationale leads with the bite: _"Client lib choice barely matters"_ / _"Prop-drilling pain is real but small"_ / _"Reducers and middleware earn their weight here"_ / _"A library is overhead you'll regret"_. No directional verbs.
- Format clean. Brevity caps hold (candidates 9–13 words, rationales 17–22 words).
- Strongest rationale: `1.`'s _"Client lib choice barely matters once a server-cache owns the hot path; picking Redux/Zustand here solves the wrong problem"_ — names the wrong-problem trap that grill-me's _"the most common mistake I see"_ also flagged.
