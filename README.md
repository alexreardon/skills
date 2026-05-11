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
- ⬇️ Every question orders recommendations in order from most to least recommended paths forward
- 💅 Leveraging consistent markdown formatting to make it easy to predictably parse outputs (super helpful when you have to answer lots of questions!)
- 🧘 What you need to do for every turn should be obvious (answer a question, provide more details, etc)

#### Compared to `/grill-me`

The same prompt run through each skill independently. Two scenarios — opening a session (clarifying intent) and picking between named options (a recommendation decision).

##### Clarifying intent

Prompt: _"I want to add rate limiting to my API"._

<table>
<tr>
<th><code>/grill-me</code> output</th>
<th><code>/cook-me</code> output</th>
</tr>
<tr>
<td valign="top">

Let's start at the top of the tree.

**Question 1: What's the primary goal of this rate limiter?**

The answer shapes almost every downstream decision (algorithm, storage, key strategy, response behavior), so I want to pin it before we go deeper.

Common goals, roughly in order of how often they're the real driver:

- **a) Abuse prevention** — stop scrapers, credential stuffing, brute force. Tends to want aggressive per-IP limits and hard blocks.
- **b) Fair-use / multi-tenant fairness** — stop one customer from starving others. Wants per-account quotas, often tiered by plan.
- **c) Cost control** — protect an expensive downstream (LLM tokens, third-party API, DB). Wants limits tied to resource consumption, not just request count.
- **d) Infrastructure protection** — keep the service from falling over under load. Often better served by load shedding / concurrency limits than classical rate limiting.
- **e) Compliance / contractual** — you've published a "100 req/min" SLA and need to enforce it.

**My recommendation: (b) fair-use, per-authenticated-account**, unless you tell me otherwise. It's the most common real need for a product API, it composes well with later additions (you can layer per-IP abuse limits on top), and it forces you to think about identity up front — which you'll need anyway.

Which of these is closest to your actual motivation? If it's a mix, which one dominates?

</td>
<td valign="top">

**`Q1 of ~7.`**
> What's driving the rate limit — abuse protection, cost control, fairness across tenants, or upstream quota?

_The candidates and what they imply:_

`1)` **Abuse / DoS protection** — block bursty or malicious clients before they degrade the service.
_Pushes toward per-IP or per-key limits with aggressive short windows and hard rejects._

`2)` **Cost control** — cap spend on paid downstream dependencies (LLM tokens, third-party APIs, egress).
_Pushes toward global or per-tenant budgets measured in units (tokens, dollars), often with daily/monthly windows._

`3)` **Fairness across tenants** — stop one customer starving others on shared capacity.
_Pushes toward per-tenant quotas with queueing or weighted fair sharing, not just rejection._

`4)` **Upstream quota compliance** — you're a client of an API with its own limits and need to stay under them.
_Pushes toward a client-side token bucket sized to the upstream contract, with backpressure to your callers._

**Recommendation:** Most teams adding rate limiting for the first time are solving `1)` — start there unless you already feel pain from one of the others.

_**Next step:** Pick a number, or describe the actual pressure you're feeling._

</td>
</tr>
</table>

##### Choosing an option

Prompt: _"I'm adding rate limiting to my API. What algorithm should I use — token bucket, leaky bucket, fixed window, or sliding window?"_

<table>
<tr>
<th><code>/grill-me</code> output</th>
<th><code>/cook-me</code> output</th>
</tr>
<tr>
<td valign="top">

**Recommendation: token bucket.**

It's the most flexible of the four — it handles steady traffic well, allows controlled bursts (which real clients almost always need), and is cheap to implement (two numbers per key: tokens remaining + last refill timestamp). Stripe, AWS, and GitHub all use it for their public APIs for these reasons.

Quick contrast for context:

- **Leaky bucket** — smooths output to a constant rate. Great for protecting a downstream system that can't handle bursts, but punishes legitimate bursty clients.
- **Fixed window** — simplest, but has the "double burst at the boundary" problem (a client can fire 2x the limit across a window edge).
- **Sliding window** — fixes the boundary problem, but costs more memory/CPU and is overkill unless you need precise enforcement.

**First question to nail down before we go further:**

Do your clients have legitimate bursty traffic patterns (e.g., a batch job that fires 50 requests then idles, or a UI that loads several resources on page open)? Or is traffic expected to be roughly steady?

**My guess:** yes, bursty — almost all real API traffic is. If so, token bucket is the right call and we can move on to sizing the bucket (capacity + refill rate). If your traffic is genuinely steady or you're specifically shielding a fragile backend, we should reconsider leaky bucket.

</td>
<td valign="top">

**`Q1 of ~7.`**
> Which algorithm should enforce your rate limits?

**`Options`** _(best first)_

`1)` **Token bucket**
_Allows short bursts up to bucket size, then steady refill rate. Most permissive for legitimate spiky traffic._

`2)` **Sliding window**
_Smooth, accurate per-interval limits without edge spikes. Heavier state per key._

`3)` **Leaky bucket**
_Enforces a strictly constant outflow rate. Best when downstream needs smoothing, not burst tolerance._

`4)` **Fixed window**
_Cheapest and simplest, but allows 2x burst at window boundaries._

**Assumption:** Your traffic has legitimate bursts (most APIs do) and you're protecting throughput, not smoothing a fragile downstream.

**Recommendation:** `1)` Token bucket — best default for an API, tolerates real-world bursts while capping sustained rate.

_**Next step:** Accept `1)`, pick another, or correct the assumption._

</td>
</tr>
</table>
