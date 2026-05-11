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

**Prompt:** `"I want to add rate limiting to my API"`

> 🧘 `/cook-me` has a consistent flow
>
> 1. One anchored question
> 2. ranked candidates with the non-obvious tradeoff under each
> 3. recommendation
> 4. explicit next step

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

`1.` **Fairness across tenants**<br>
One tenant's burst shouldn't degrade latency for others sharing the pool.<br>
_Forces per-tenant keys and usually a token bucket; a global limit "feels fair" but actually lets the loudest tenant consume the whole budget right up to the ceiling._

`2.` **Cost control**<br>
Cap spend on a downstream paid dependency (LLM tokens, third-party API, egress).<br>
_The limit must track the cost unit, not requests — one request can be 100x another, so a request/sec cap leaks budget while looking healthy on dashboards._

`3.` **Abuse / credential stuffing**<br>
Stop scrapers, brute-force login, scripted enumeration.<br>
_Per-IP breaks behind NAT and CGNAT (mobile carriers share IPs across thousands of users); you'll need per-account or per-session keys too, and the 429 itself becomes a signal attackers probe._

`4.` **Protect the service from overload**<br>
Shed load before the database or worker pool tips over.<br>
_This is a load-shedding problem, not a rate-limit problem — a fixed RPS cap set for steady state will either throttle legitimate traffic during normal spikes or fail to engage when the real bottleneck (DB connections, queue depth) saturates first._

**Recommendation:** Pick the one that would make you ship this week; the others can layer on later with the same infrastructure.

_**Next step:** name the primary driver._

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

**`Q1 of ~2.`**
> What are you protecting against?

**`Candidates`** _(↓ most to least promising)_

`1.` **Backend overload from bursty clients**<br>
Your DB, downstream service, or worker pool falls over under spikes. Limit exists to keep infra alive.<br>
_Burst tolerance matters more than fairness — a strict limit that rejects legitimate bursts costs more than it saves._<br>
`Leads to →` token bucket (absorbs retry storms without dropping legit traffic)

`2.` **Per-user abuse / fairness across tenants**<br>
One account scraping or looping shouldn't degrade others. Limit is per-identity, not global.<br>
_Boundary-reset exploits (1000 req at 11:59:59, 1000 more at 12:00:00) are the actual failure mode here, not raw throughput._<br>
`Leads to →` sliding window per-account (closes the boundary-reset exploit)

`3.` **Cost control on a paid downstream (LLM, SMS, geocoding)**<br>
Each call costs real money. Limit caps spend, not load.<br>
_Rate-limiting is the wrong primitive — you want a budget/quota with hard cutoff, because spend doesn't recover when the window rolls over._<br>
`Leads to →` budget/quota with circuit breaker (rate limit is the wrong tool)

`4.` **Compliance / contractual SLA enforcement**<br>
"Free tier: 100 req/day" is in the contract. Limit is a billing boundary.<br>
_Algorithm is irrelevant; correctness of the counter (durable, survives restarts, no double-count across replicas) is the whole problem._<br>
`Leads to →` durable counter in Redis/DB (algorithm choice doesn't matter)

**Recommendation:** `1.` — most "I'm adding rate limiting" questions are really "my backend is getting hammered."

_**Next step:** pick a candidate or name a different threat._

---

**User picks `1.` Backend overload from bursty clients**

---

**`Q2 of ~2.`**
> Which algorithm for absorbing bursts while protecting the backend?

**`Options`** _(↓ most to least recommended)_

`1.` **Token bucket**<br>
Refill at steady rate, burst up to bucket size. Clients can spend accumulated tokens for legitimate spikes.<br>
_Survives retry storms gracefully — when downstream recovers, the bucket has refilled and clients drain naturally, instead of synchronizing on a window boundary and re-DDoSing you._

`2.` **Sliding window (log or weighted)**<br>
Track requests over a rolling interval; smooth, no boundary effects.<br>
_Memory cost scales with request volume per key (log variant) or you accept approximation error (weighted variant) — fine for per-user limits, expensive for a global high-QPS limit._

`3.` **Leaky bucket**<br>
Queue requests, drain at fixed rate. Smooths output completely.<br>
_Adds latency by design — requests wait in the queue. Wrong choice if your backend can handle bursts at all, because you're rate-limiting yourself harder than necessary._

`4.` **Fixed window**<br>
Counter resets every N seconds. Simplest to implement.<br>
_Boundary exploit: a client can fire 2x the limit by stacking requests at the window edge. Also synchronizes retry storms across all clients at reset time — exactly the failure mode you're trying to prevent._

**Assumption:** your backend can handle short bursts (it's sustained load that kills it), so you want burst tolerance not smoothing.

**Assumption:** you're storing state in Redis or similar — token bucket is `INCR` + `EXPIRE` + a timestamp, not meaningfully harder than fixed window.

**Recommendation:** `1.` token bucket.

_**Next step:** accept `1.`, pick another, or correct an assumption._

</td>
</tr>
</table>

</details>
