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
> What's the primary access pattern your product is built around?

**`Candidates`** _(↓ most to least promising)_

`1.` **Relational + transactional**<br>
Multi-entity joins, invoices, audit trails, reporting across customer accounts.<br>
_Schema drift bites hardest in B2B; rigid constraints save you from messy customer-specific edge cases later._<br>
`Leads to →` Postgres

`2.` **Document-shaped per tenant**<br>
Each customer stores nested, schema-flexible blobs you rarely join across.<br>
_Cross-tenant analytics becomes painful later; flexibility upfront trades against reporting pain at scale._<br>
`Leads to →` Postgres JSONB, MongoDB

`3.` **Key-value at predictable scale**<br>
Known access keys, single-digit-ms reads, write volumes that would crush a relational primary.<br>
_Ad-hoc queries become impossible; every new access pattern forces a schema or index migration._<br>
`Leads to →` DynamoDB

`4.` **Unsure / early-stage**<br>
You don't yet know the dominant query shape because the product isn't built.<br>
_Premature optimization bites worst here; picking the wrong specialist locks you out of pivots cheaply._<br>
`Leads to →` Postgres (default)

**Recommendation:** `1.` — most B2B SaaS is relational under the hood (accounts, users, billing, audit), and Postgres handles document workloads via JSONB if you need flexibility later.

_**Next step:** Pick a candidate, or describe the dominant query shape and I'll place it._

---

## Notes

- Format clean across all 4 candidates. Brevity caps hold.
- Every rationale leads with the bite: _"Schema drift bites hardest in B2B"_ / _"Cross-tenant analytics becomes painful later"_ / _"Ad-hoc queries become impossible"_ / _"Premature optimization bites worst here"_.
- New `Leads to →` rule exercised on `2.` (`Postgres JSONB, MongoDB` — plural). Other candidates land naturally on a single destination, which the rule still allows.
- Recommendation line is branch-aware (notes that JSONB folds in `2.`'s flex case under `1.`).
