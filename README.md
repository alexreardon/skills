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

_The opinions_

- 1️⃣ Only one question per turn. Never ask multiple questions at a time
- 🙌 Questions and options are phrased in positive language so it's easy to say "yes" to things (no double negatives)
- ⬇️ Every question orders proposals and options in order from most to least recommended paths forward
- 💅 Leveraging consistent markdown formatting to make it easy to predictably parse outputs (super helpful when you have to answer lots of questions!)
- 🧘 What you need to do for every turn should be obvious (answer a question, provide more details, etc)

#### `/grill-me` vs `/cook-me` output

To better understand what `/cook-me` does, below is listed out a output comparison between `/grill-me` and `/cook-me`. All below output was captured with `Opus 4.7 (Anthropic)`.

#### Model asking clarifying questions

**Prompt**:
> `"I want to add rate limiting to my API"`

_What `/cook-me` brings_

- 🎯 Candidates are numbered and explicitly ordered (no guessing which is the recommended path)
- 🪜 3 lines per candidate: the name, what it is, then why it matters (easy to skim, easy to compare)
- 💡 Rationales call out the non-obvious gotcha, not generic "pushes toward X" template language
- 👉 The final `**Next step:**` line makes the action obvious — no wondering what to do

<details>
<summary>See the full output comparison</summary>

<br>

<table>
<tr>
<th width="50%"><code>/grill-me</code> output</th>
<th width="50%"><code>/cook-me</code> output</th>
</tr>
<tr>
<td valign="top">

Before we dive into the design tree, I need to anchor on the *why* — every downstream decision (algorithm, storage, granularity, failure mode) cascades from this.

**What's the primary problem you're trying to solve with rate limiting?**

A few common motivations, and they pull the design in very different directions:

1. **Protecting infrastructure from overload** — preventing a traffic spike (organic or accidental) from taking down your service. Usually points toward coarse, global limits and aggressive shedding.
2. **Preventing abuse / DoS** — stopping malicious actors from hammering endpoints. Usually points toward per-IP or per-credential limits with hard blocks and possibly escalating penalties.
3. **Enforcing fair usage between tenants** — making sure one noisy customer doesn't degrade experience for others. Usually points toward per-account/per-key quotas with isolation guarantees.
4. **Monetization / plan tiers** — free vs. paid users get different ceilings. Usually points toward per-customer quotas tied to billing, with clear 429 semantics and upgrade paths.
5. **Protecting a downstream dependency** — e.g., you call a third-party API with its own limits and need to stay under them. Points toward outbound rate limiting, not inbound.

**My recommendation:** start by picking the *single* dominant motivation. Most teams try to solve all five with one system and end up with something that does none well. If more than one applies, rank them — we'll design for the top one and check the others as constraints, not co-equal goals.

Which of these is closest to your situation? Or is it something else?

</td>
<td valign="top">

**`Q1 of ~4.`**
> What's driving the rate limit — abuse prevention, cost control, or fairness across tenants?

**`Candidates`** _(↓ most to least promising)_

`1)` **Fairness across tenants**<br>
One tenant's burst shouldn't degrade latency for others sharing the pool.<br>
_Forces per-tenant keys and usually a token bucket; a global limit "feels fair" but actually lets the loudest tenant consume the whole budget right up to the ceiling._

`2)` **Cost control**<br>
Cap spend on a downstream paid dependency (LLM tokens, third-party API, egress).<br>
_The limit must track the cost unit, not requests — one request can be 100x another, so a request/sec cap leaks budget while looking healthy on dashboards._

`3)` **Abuse / credential stuffing**<br>
Stop scrapers, brute-force login, scripted enumeration.<br>
_Per-IP breaks behind NAT and CGNAT (mobile carriers share IPs across thousands of users); you'll need per-account or per-session keys too, and the 429 itself becomes a signal attackers probe._

`4)` **Protect the service from overload**<br>
Shed load before the database or worker pool tips over.<br>
_This is a load-shedding problem, not a rate-limit problem — a fixed RPS cap set for steady state will either throttle legitimate traffic during normal spikes or fail to engage when the real bottleneck (DB connections, queue depth) saturates first._

**Recommendation:** Pick the one that would make you ship this week; the others can layer on later with the same infrastructure.

_**Next step:** name the primary driver._

</td>
</tr>
</table>

</details>

#### Model providing recommendations

**Prompt**:
> `"I'm adding rate limiting to my API. What algorithm should I use — token bucket, leaky bucket, fixed window, or sliding window?"`

_What `/cook-me` brings_

- 🛑 Refuses to rank options when intent isn't pinned — runs a clarifying exploration turn first, then the recommendation
- 🔄 Multi-turn flow keeps each turn focused on one decision (the user's reply between turns is shown explicitly)
- 🎚️ Stacks `**Assumption:**` lines for the guessable defaults the recommendation rides on — you can correct any of them in-place
- 🔍 Each option's rationale names the specific operational gotcha (not the textbook contrast you'd read in any rate-limiting blog post)

<details>
<summary>See the full output comparison</summary>

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

**`Q1 of ~3.`**
> What are you protecting — and from what?

**`Candidates`** _(↓ most to least promising)_

`1)` **Expensive backend (DB, downstream API, LLM tokens)**<br>
You're shielding a resource with real per-call cost or contention.<br>
_Cost-per-request makes burst tolerance dangerous — one user's "smooth" burst is your pager at 3am when the pool saturates._

`2)` **Abuse / credential stuffing / scraping**<br>
You're stopping adversaries, not smoothing legitimate traffic.<br>
_Per-IP keying gets defeated by botnets in a weekend; the algorithm matters less than the identity dimension you key on._

`3)` **Fair-share between tenants on shared infra**<br>
Noisy-neighbor isolation across customers on one cluster.<br>
_Global limits give the wrong answer here — a quiet tenant gets throttled because a loud one filled the window._

`4)` **Compliance / contractual quota (e.g., "10k req/day per plan")**<br>
Billing or SLA enforcement, not protection.<br>
_This is accounting, not rate limiting — durability and audit trail matter more than algorithm choice._

**Recommendation:** `1)` — most common reason teams reach for rate limiting, and the algorithm choice genuinely hinges on it.

_**Next step:** pick one, or describe the threat in your own words._

---

**User picks `1)` Expensive backend (DB, downstream API, LLM tokens)**

---

**`Q2 of ~3.`**
> Which algorithm for protecting an expensive backend?

**`Options`** _(↓ most to least recommended)_

`1)` **Token bucket**<br>
Refill at steady rate, allow bursts up to bucket size; reject when empty.<br>
_The bucket size is a knob you can tune per-route without redeploying limits — and crucially, it absorbs retry storms after a brief outage instead of amplifying them like fixed-window would at the boundary._

`2)` **Sliding window (log or weighted)**<br>
Track requests over a rolling time range.<br>
_Accurate, but the log variant stores a timestamp per request in Redis — at 10k RPS per key that's a hot key with O(N) trimming; the weighted-counter approximation is what people actually ship._

`3)` **Leaky bucket (queue)**<br>
Requests queue and drain at fixed rate; overflow rejects.<br>
_Smooths perfectly but adds latency by design — your p99 now includes queue wait, which is invisible in your "requests per second" dashboards and shows up as mystery slowness downstream._

`4)` **Fixed window**<br>
Count per calendar minute/hour, reset at boundary.<br>
_The boundary-doubling problem is real: a client can fire 2x your limit in 2 seconds straddling the reset — exactly the burst your expensive backend can't handle._

**Assumption:** You're keying per-user (or per-API-key), not globally — global limits on an expensive backend usually mean you bought the wrong size backend.

**Assumption:** You're running this in Redis or similar shared store, not in-process — in-process limiters silently allow N × replicas of your stated limit.

**Recommendation:** `1)` Token bucket — burst tolerance plus simple Redis primitives (`INCR` + TTL or Lua script), and it's what every major API gateway ships by default for a reason.

_**Next step:** accept `1)`, pick another, or correct an assumption._

</td>
</tr>
</table>

</details>
