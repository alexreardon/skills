# 06. Polyrepo → monorepo decision

**Prompt:**
> "We have 12 services in separate repos. Should we move to a monorepo?"

---

## `/grill-me` output

What's driving the move to a monorepo right now - is there a specific pain point (e.g. cross-service changes, dependency drift, duplicated tooling, slow onboarding), or is it more of a general "this feels right" intuition?

My recommendation: identify the single biggest concrete pain point first. "We have 12 repos" isn't a reason; "every shared library bump takes 12 PRs and 3 days" is. Without a specific pain, a monorepo migration is months of work for marginal gain - and often introduces new pains (CI scale, tooling complexity, blast radius).

---

## `/cook-me` output

**`Q1 of ~6.`**
> What's driving the monorepo question — pain you're hitting today, or a forward-looking bet?

**`Candidates`** _(↓ most to least promising)_

`1.` **Cross-service change friction**<br>
Shipping a feature touches 3+ repos; PRs land out of order, versions drift.<br>
_Coordination cost is the bite; monorepo collapses it but forces shared tooling decisions._<br>
`Leads to →` atomic commits, shared CI, version pinning

`2.` **Shared code duplication**<br>
Same auth/logging/types copy-pasted across services, drifting independently.<br>
_Drift causes silent prod bugs; a shared lib in one repo is the fix, not necessarily a monorepo._<br>
`Leads to →` shared lib, published packages, monorepo

`3.` **Onboarding and discoverability**<br>
New engineers can't find the service that owns X; 12 READMEs, 12 setups.<br>
_Cognitive load compounds per service; a monorepo fixes browsing but not architectural sprawl._<br>
`Leads to →` monorepo, service catalog, docs portal

`4.` **Forward bet on scale**<br>
No acute pain; you've read Google/Meta posts and want the option value early.<br>
_Migration cost lands now for benefits that may never arrive; sunk-cost lock-in is the real risk._<br>
`Leads to →` defer, spike, status quo

**Recommendation:** Start with `1.` — cross-service change friction is the most common real driver and the one a monorepo most directly fixes; if it's actually `4.`, the answer is usually "don't."

_**Next step:** Pick the driver, or describe the pain in your own words._

---

## Notes

- Format clean across all 4 candidates. Brevity caps hold.
- Every rationale leads with the bite: _"Coordination cost is the bite"_ / _"Drift causes silent prod bugs"_ / _"Cognitive load compounds per service"_ / _"Migration cost lands now for benefits that may never arrive"_.
- New `Leads to →` rule fully exercised — all four candidates use plural destinations. `2.` and `3.` deliberately route AWAY from monorepo in the destinations themselves (`shared lib, published packages, monorepo` and `monorepo, service catalog, docs portal`) — the user sees cheaper alternatives at the same scan depth as the heavy migration.
- `1.`'s destination (`atomic commits, shared CI, version pinning`) is at ~42 chars — slightly over the soft cap. Reads fine but worth flagging as drift if it persists across passes.
- Anti-monorepo bias is structural: 3 of 4 candidates name a cheaper non-monorepo fix in either the rationale or the destination list. The Recommendation line summarizes: _"if it's actually `4.`, the answer is usually 'don't.'"_
