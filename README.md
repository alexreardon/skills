# `alexreardon/skills`

[Alex Reardon's](https://x.com/alexandereardon) collection of [agent skills](https://vercel.com/docs/agent-resources/skills).

## Install

```bash
npx skills add alexreardon/skills
```

To install a single skill from this repo:

```bash
npx skills add alexreardon/skills --skill cook-me
```

> [More installation options](https://github.com/vercel-labs/skills)

## Skills

### [`/cook-me`](skills/cook-me/SKILL.md) 🧑‍🍳

Opinionated version of the [`/grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) skill by [Matt Pocock](https://x.com/mattpocockuk) that makes `/grill-me` easier to work with through consistent patterns for question asking, language and layout.

_`/cook-me` opinions_

- 1️⃣ Only one question per "turn" (ai output and user response). Never ask multiple questions at a time
- 🪣 Put "exploration" and "option selection" in their own turns
- 🙌 Options phrased in positive language so it's easy to say "yes" to things (no double negatives)
- ⬇️ Every turn orders proposals and options in order from most to least recommended paths forward
- 💅 Consistent layout and formatting to make it easy to predictably parse outputs (super helpful when you have to answer lots of questions!)
- 🧘 What you need to do for every turn should be obvious (answer a question, provide more details, etc)

#### `/grill-me` vs `/cook-me` output

**Prompt:** `"I'm building a React app and trying to choose a state management library. Should I use Redux Toolkit, Zustand, Jotai, or React Context?"`

> 🧘 `/cook-me` keeps the same shape every turn, so options are easy to scan and pick

<details>
<summary>Show output comparison</summary>

<br>

<table>
<tr>
<th width="50%"><code>/grill-me</code> output</th>
<th width="50%"><code>/cook-me</code> output</th>
</tr>
<tr>
<td valign="top">

Before picking a library, the most load-bearing question is about the shape of your state itself, because that determines which tool actually fits.

**What kind of state dominates your app?**

a) Mostly **server state** (data fetched from APIs, cached, refetched) with a small amount of local UI state
b) Mostly **client state** (complex local logic, derived values, cross-component coordination) — e.g. a builder, editor, or dashboard with rich interactions
c) Mostly **simple shared UI state** (theme, auth user, modal open/closed, a few global flags)
d) A genuinely large, deeply nested **normalized domain model** (entities with relationships, lots of updates across many slices)

My recommendation: **(a)** — for the vast majority of React apps today, what people call "state management" is actually server state, and the right answer is to pair a server-state library (React Query / TanStack Query or RTK Query) with a lightweight client store only where needed. Reaching for Redux Toolkit / Zustand / Jotai before answering this question is the most common mistake I see, because each of those libraries shines in a different one of these four scenarios.

Which one is yours?

</td>
<td valign="top">

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

</td>
</tr>
</table>

</details>

<br>

**Prompt:** `"I'm adding rate limiting to my API. What algorithm should I use — token bucket, leaky bucket, fixed window, or sliding window?"`

> 👨‍🍳 `/cook-me` breaks down the interaction into two distinct turns.
>
> - Q1 is to understand the intent of the change
> - Q2 is to choose an algorithm based on that intent

<details>
<summary>Show output comparison</summary>

<br>

<table>
<tr>
<th width="50%"><code>/grill-me</code> output</th>
<th width="50%"><code>/cook-me</code> output</th>
</tr>
<tr>
<td valign="top">

**Recommendation: sliding window counter** (or token bucket if you need to allow bursts).

Quick contrast:
- **Fixed window** — simplest, but allows 2x burst at window boundaries (e.g., 100 req at 12:00:59 + 100 req at 12:01:00). Reject unless precision doesn't matter.
- **Sliding window (log)** — most accurate, but stores every request timestamp. Expensive at scale.
- **Sliding window (counter)** — approximates sliding-log using two fixed-window counters and a weighted average. Cheap, accurate enough, my default for HTTP APIs.
- **Token bucket** — best when you want to *allow* controlled bursts (e.g., "100 req/sec sustained, but 500 req burst is fine"). Common for SDKs and internal services.
- **Leaky bucket** — enforces a *smooth* output rate (queue + constant drain). Right for protecting a downstream that can't tolerate spikes; wrong if you just want to count requests.

The choice hinges on what you're actually protecting and how clients behave. So:

**Question 1: What are you rate-limiting and why?**

Concretely:
- Is this protecting a specific downstream (DB, third-party API, expensive compute) from being overwhelmed, or is it fairness/abuse-prevention across tenants?
- Per-user, per-IP, per-API-key, or global?
- Do legitimate clients send bursty traffic (e.g., a UI that fires 10 requests on page load) or steady traffic?

My guess: it's per-API-key abuse prevention for an HTTP API with somewhat bursty legit clients — in which case **token bucket** wins because it forgives natural bursts while capping sustained rate. But tell me the actual shape and I'll either confirm or switch the recommendation.

</td>
<td valign="top">

**`Q1 of ~5.`**
> What are you actually protecting against? Different threats pick different algorithms.

**`Candidates`** _(↓ most to least promising)_

`1.` **Abusive bursts from individual clients**<br>
One client hammers your API, starving everyone else.<br>
_Per-key fairness becomes the dominant concern; global throughput is fine._<br>
`Leads to →` token bucket, sliding window

`2.` **Backend capacity protection**<br>
A downstream DB or service has a hard concurrency ceiling.<br>
_Aggregate cap matters more than per-client fairness; queueing semantics start to dominate._<br>
`Leads to →` leaky bucket, concurrency limiter

`3.` **Cost control on metered dependencies**<br>
Each request burns paid quota (LLM tokens, third-party API calls).<br>
_Hard monthly ceilings beat smooth shaping; overage is a billing event, not latency._<br>
`Leads to →` fixed window, quota counter

`4.` **Brute-force / credential stuffing**<br>
Attackers probing auth endpoints with stolen credentials.<br>
_False negatives leak accounts; per-IP+per-account compound keys matter more than algorithm choice._<br>
`Leads to →` sliding window, exponential backoff

**Recommendation:** `1.` — most public APIs start here, and it's the only driver that meaningfully constrains the four algorithms you listed.

_**Next step:** Pick the driver, or name a different threat I missed._

---

**User picks `1.` Abusive bursts from individual clients**

---

**`Q2 of ~5.`**
> Given per-client fairness is the goal, which algorithm?

**`Options`** _(↓ most to least recommended)_

`1.` **Token bucket**<br>
Each client accrues tokens at a steady rate up to a cap; requests spend tokens.<br>
_Legitimate bursts pass through cleanly; the burst cap is a second knob you have to tune and defend._

`2.` **Sliding window (log or counter)**<br>
Tracks request timestamps in a rolling window per client.<br>
_Memory grows with request volume per key; accurate but the priciest to run at scale._

`3.` **Leaky bucket**<br>
Requests queue and drain at a fixed rate; overflow is rejected.<br>
_Bursts get smoothed into latency instead of rejection; clients see slow responses, not 429s._

`4.` **Fixed window**<br>
Counter per client resets every N seconds.<br>
_Boundary-crossing bursts let clients send 2x the limit in a two-second span; cheapest but leakiest._

**Assumption:** Your traffic has legitimate bursts (most user-facing APIs do) and you're willing to run Redis or similar for shared counter state.

**Recommendation:** `1.` — token bucket absorbs real bursts without punishing well-behaved clients, and the two knobs (rate, burst) map cleanly to the limits you'll publish.

_**Next step:** Accept `1.`, pick another, or correct the assumption._

</td>
</tr>
</table>

</details>
