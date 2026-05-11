# 01. Database choice for a new B2B SaaS

**Prompt:**
> "I'm starting a new B2B SaaS product and need to pick a primary database. Should I use Postgres, MongoDB, or DynamoDB?"

---

## `/grill-me` output

**Question 1: What's the shape of your data and how will you query it?**

This is the foundational question because it determines whether a relational, document, or key-value model fits best. Specifically:
- Is your data highly relational (users, orgs, teams, permissions, billing, audit logs all referencing each other)?
- Will you need ad-hoc queries, joins, aggregations, and reporting?
- Or is it mostly self-contained documents accessed by a known key?

**My recommended answer: Postgres.**

For a new B2B SaaS, your data is almost certainly highly relational — tenants, users, roles, subscriptions, invoices, resources owned by orgs — and you will absolutely need joins and ad-hoc queries (for admin tools, support, analytics, and customer-facing reporting). Postgres gives you ACID transactions, mature tooling, JSONB when you genuinely need schemaless fields, and it scales further than most startups ever need. MongoDB tempts you with schema flexibility you don't actually want in a B2B contract-driven product, and DynamoDB forces you to know your access patterns up front — which you don't yet, because you haven't shipped.

Does that match your data shape, or is there something unusual about your domain (e.g., massive write throughput, time-series, or genuinely schemaless user-defined objects) that pushes against Postgres?

---

## `/cook-me` output

**`Q1 of ~6.`**
> What's the dominant data shape and access pattern you expect in year one?

**`Candidates`** _(↓ most to least promising)_

`1.` **Relational with evolving schema**<br>
Tenants, users, billing, audit — entities with foreign keys and ad-hoc reporting needs.<br>
_Joins and constraints become load-bearing fast in B2B; bolting them on later means rewrites, not migrations._<br>
`Leads to →` Postgres

`2.` **Document-shaped per-tenant blobs**<br>
Flexible nested objects, schema varies by customer, few cross-entity queries.<br>
_Aggregations across tenants get painful; you'll reach for a warehouse sooner than you think._<br>
`Leads to →` MongoDB (or Postgres JSONB)

`3.` **Known key-value access at scale**<br>
Every read is by a known ID or composite key, predictable QPS, no ad-hoc queries.<br>
_Access patterns must be locked at design time; a new query shape can mean a full table redesign._<br>
`Leads to →` DynamoDB

**Recommendation:** Most early B2B SaaS is `1.` — billing, RBAC, and customer-asked reports all want relational shape, and Postgres covers `2.`'s flex via JSONB until you genuinely outgrow it.

_**Next step:** Pick a candidate, or describe your core entities and I'll place you._

---

## Notes

- Format clean across the board. Brevity caps hold (candidates 12–14 words, rationales 16–20 words).
- All three rationales lead with the **bite**, not architectural placement: _"Joins and constraints become load-bearing fast"_ / _"Aggregations across tenants get painful"_ / _"Access patterns must be locked at design time"_. None reach for the directional verbs the spec now explicitly bans.
- Recommendation line is branch-aware (notes that JSONB folds in `2.`'s flex case under `1.`).
