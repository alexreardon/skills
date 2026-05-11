# 03. Caching strategy for a web application

**Prompt:**
> "I want to add caching to my web application. Where should I cache and how should I handle invalidation?"

---

## `/grill-me` output

What kind of data are you planning to cache first? My recommendation: start with read-heavy, expensive-to-compute responses (e.g. database query results for list/detail endpoints) rather than full HTML pages or static assets — those give the highest hit rate per unit of complexity, and they surface the invalidation questions early so you can design the rest of the strategy around them.

---

## `/cook-me` output

**`Q1 of ~6.`**
> Before ranking cache layers, what problem are you solving — latency, backend load, or cost?

**`Candidates`** _(↓ most to least promising)_

`1.` **Read latency**<br>
P95/P99 tail latency on hot reads is the felt pain; users notice slow pages.<br>
_Stale reads become tolerable in exchange for speed; that's the deal you're signing up for._<br>
`Leads to →` edge cache, CDN, in-memory (+ more)

`2.` **Backend / DB load**<br>
Origin or database is saturating under repeated identical reads; you need to absorb traffic.<br>
_Thundering herds on miss become the dominant failure mode; single-flight and TTL jitter stop being optional._<br>
`Leads to →` Redis, read-replica, request coalescing

`3.` **Compute / egress cost**<br>
Recomputation or per-request egress is the budget line item driving the project.<br>
_Hit-rate directly determines ROI; a 60% hit cache may not pay for its own ops overhead._<br>
`Leads to →` CDN, materialized views

`4.` **Personalized / per-user reads**<br>
Hot paths are user-scoped (feeds, dashboards), so shared caches barely help.<br>
_Cache keys explode by user; memory budget and eviction policy decide whether this works at all._<br>
`Leads to →` per-user Redis, session cache

**Recommendation:** `1.` — latency is the most common real driver and pins which layer (edge vs app vs DB) actually helps; the others reshape the answer enough that ranking layers now would be premature.

_**Next step:** Pick the dominant driver, or tell me it's a mix and which leads._

---

## Notes

- **Caching was the prompt that exposed the directional-language trap across passes 4–7** (rationales kept leading with location words like _"Pushes cache toward the edge"_). Pass 8's worked WRONG → RIGHT example fixed it; this pass holds the line — all four rationales lead with the bite: _"Stale reads become tolerable"_ / _"Thundering herds on miss become the dominant failure mode"_ / _"Hit-rate directly determines ROI"_ / _"Cache keys explode by user"_.
- **New `Leads to →` rule fully exercised.** All 4 candidates use plural destinations; `1.` even uses `(+ more)` for overflow (`edge cache, CDN, in-memory (+ more)`). Caching is a layered space — there's almost never one right cache — so plural reads more honestly here than singular.
- Branch-aware Recommendation: pins `1.` while noting `2.` would push you to opposite ends of the stack — primes the user that "where" follows from "why."
- Format clean across all 4 candidates. Brevity caps hold.
