# 03. Caching strategy for a web application

**Prompt:**
> "I want to add caching to my web application. Where should I cache and how should I handle invalidation?"

---

## `/grill-me` output

What kind of data are you planning to cache first? My recommendation: start with read-heavy, expensive-to-compute responses (e.g. database query results for list/detail endpoints) rather than full HTML pages or static assets — those give the highest hit rate per unit of complexity, and they surface the invalidation questions early so you can design the rest of the strategy around them.

---

## `/cook-me` output

**`Q1 of ~8.`**
> What problem are you trying to solve with caching?

**`Candidates`** _(↓ most to least promising)_

`1.` **Read latency**<br>
Pages or API responses feel slow; you want sub-100ms responses for hot paths.<br>
_Staleness becomes the dominant tradeoff; reads are easy to cache but invalidation is where you'll bleed._<br>
`Leads to →` edge/CDN or in-process cache

`2.` **Database load**<br>
Origin DB is saturated or expensive; you want to absorb repeated identical queries.<br>
_Thundering herds on cache miss can crater the DB worse than no cache; needs request coalescing._<br>
`Leads to →` Redis/Memcached in front of DB

`3.` **Expensive computation**<br>
Specific results (rendered pages, ML inference, aggregations) cost CPU/$ to produce.<br>
_Cache key design becomes the whole game; wrong key granularity either bloats storage or never hits._<br>
`Leads to →` memoization layer keyed on inputs

`4.` **Third-party API cost or rate limits**<br>
You're paying per call or hitting quotas on an upstream you don't control.<br>
_Their TTL guidance (or lack of it) dictates yours; you inherit their freshness contract whether stated or not._<br>
`Leads to →` keyed response cache with vendor-aligned TTL

**Recommendation:** Name the bottleneck before picking a layer — `1.` and `2.` look similar but push you to opposite ends of the stack (edge vs. origin-adjacent), and invalidation strategy follows from that pick, not the other way around.

_**Next step:** Pick the driver, or describe the symptom in your own words._

---

## Notes

- **Caching was the prompt that exposed the directional-language trap across passes 4–7.** Pass 4: _"Pushes cache toward the edge"_. Pass 5: _"Points you at edge or in-process caches"_. Pass 6: _"Shifts the conversation to shared caches"_. Pass 7: _"Pulls cache close to the read path"_ + _"Pushes state to client/edge"_. The model kept dodging the explicit verb list with synonyms.
- **Pass 8: fully clean.** Every rationale now leads with the **bite**: _"Staleness becomes the dominant tradeoff"_ / _"Thundering herds on cache miss can crater the DB worse than no cache"_ / _"Cache key design becomes the whole game"_ / _"Their TTL guidance dictates yours"_. The location appears in `Leads to →` (where it belongs), not as the rationale's lead.
- The fix that landed: a worked WRONG → RIGHT example in the `Before submitting` checklist showing how the same information can be reordered so the bite leads. Verb-list bans alone weren't enough.
- Format clean across all 4 candidates. Brevity caps hold.
- Branch-aware Recommendation: _"`1.` and `2.` look similar but push you to opposite ends of the stack"_ — names the trap before pinning a pick.
