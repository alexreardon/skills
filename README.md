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

**`Q1 of ~8.`**
> What's driving the need — abuse prevention, cost control, or fairness across clients?

The candidates and what they imply:

`1)` **Abuse / DoS protection**<br>
Block bursts from a single bad actor before they reach expensive handlers.<br>
_Pushes toward per-IP or per-token limits with aggressive short windows; fail-closed on the edge (gateway/CDN) is usually right._

`2)` **Cost control on downstream resources**<br>
Cap aggregate load on a DB, paid third-party API, or LLM provider.<br>
_Pushes toward global or per-resource concurrency limits, not per-client; the limit lives near the expensive call, not the edge._

`3)` **Fairness across tenants / plan enforcement**<br>
Stop one customer from starving others, and align usage with billing tiers.<br>
_Pushes toward per-API-key quotas with longer windows (per-minute, per-day), tier-aware config, and clear `429` headers so clients can self-pace._

`4)` **Belt-and-suspenders — all of the above**<br>
You want some of each because the API is public and monetized.<br>
_Realistic for most production APIs, but the layers stack differently and the order matters; worth naming so we sequence the design instead of conflating them._

**Recommendation:** `3)` if this API has authenticated clients on plans; `1)` if it's public and unmonetized. Most teams end up at `4)` but should still pick the primary driver to anchor the design.

_**Next step:** Pick the primary driver, or tell me more about who calls this API and what hurts when they call it too much._

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

**`Q1 of ~6.`**
> Which rate-limiting algorithm fits your API?

**`Options`** _(best first)_

`1)` **Token bucket**<br>
Tokens refill at a steady rate up to a cap; each request spends one. Bursts allowed up to bucket size.<br>
_Handles bursty real-world traffic well, cheap to implement, widely supported in Redis/nginx/cloud gateways. Slightly looser than strict per-window limits._

`2)` **Sliding window (log or counter)**<br>
Tracks requests across a rolling time window, either by timestamp log or weighted counter across two fixed windows.<br>
_Most accurate limit enforcement, no edge-of-window burst exploit. Higher memory (log) or moderate complexity (counter); overkill unless you need precise fairness._

`3)` **Leaky bucket**<br>
Requests queue and drain at a fixed rate; overflow is rejected. Smooths output to a constant flow.<br>
_Great when the downstream needs steady pacing (e.g. shaping outbound calls). Rejects bursts even when capacity exists, and queueing adds latency — wrong default for user-facing APIs._

`4)` **Fixed window**<br>
Counter per clock-aligned window (e.g. per minute); resets at boundary.<br>
_Trivial to build and reason about. Allows 2x burst at window boundaries, which is usually unacceptable for abuse protection._

**Assumption:** traffic is bursty (human/client-driven), not a steady machine-to-machine stream.

**Recommendation:** `1)` token bucket — best balance of burst tolerance, simplicity, and ecosystem support.

_**Next step:** accept, pick another, or correct the assumption._

</td>
</tr>
</table>
