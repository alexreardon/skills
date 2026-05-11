# 06. Polyrepo → monorepo decision

**Prompt:**
> "We have 12 services in separate repos. Should we move to a monorepo?"

---

## `/grill-me` output

What's driving the move to a monorepo right now - is there a specific pain point (e.g. cross-service changes, dependency drift, duplicated tooling, slow onboarding), or is it more of a general "this feels right" intuition?

My recommendation: identify the single biggest concrete pain point first. "We have 12 repos" isn't a reason; "every shared library bump takes 12 PRs and 3 days" is. Without a specific pain, a monorepo migration is months of work for marginal gain - and often introduces new pains (CI scale, tooling complexity, blast radius).

---

## `/cook-me` output

**`Q1 of ~8.`**
> What's driving the monorepo question — what pain are you trying to fix?

**`Candidates`** _(↓ most to least promising)_

`1.` **Cross-service change friction**<br>
Shipping a feature touches multiple repos; PRs, versioning, and release coordination drag.<br>
_Atomic commits become the prize; without them a monorepo just relocates the same coordination cost._<br>
`Leads to →` Bazel/Nx/Turborepo + shared CI

`2.` **Shared code drift**<br>
Duplicated libs, types, or contracts diverge across services and break integrations.<br>
_A shared package boundary is the real fix; monorepo is one delivery vehicle, not the only one._<br>
`Leads to →` shared packages (mono OR published internal libs)

`3.` **Dev environment friction**<br>
Cloning 12 repos, juggling versions, and spinning up locally is painful for new and existing devs.<br>
_Tooling (devcontainers, meta-repo) often dissolves this without restructuring source control._<br>
`Leads to →` meta-repo or devcontainer, not a true monorepo

`4.` **Org/ownership confusion**<br>
Unclear who owns what; discovery and review routing is ad-hoc across 12 repos.<br>
_CODEOWNERS and a service catalog usually outperform a repo merge here; consolidation can blur ownership further._<br>
`Leads to →` catalog + CODEOWNERS, defer monorepo

`5.` **CI/release fragmentation**<br>
12 pipelines, 12 release processes, inconsistent quality gates.<br>
_Pipeline templating (reusable workflows) fixes most of this without a merge; monorepo CI brings its own selective-build problem._<br>
`Leads to →` shared CI templates

**Recommendation:** Pin the dominant pain before ranking solutions — monorepo is a heavy migration that only pays off for `1.` and `2.`; `3.`–`5.` have cheaper fixes.

_**Next step:** Pick the candidate that hurts most, or describe the actual pain in your own words._

---

## Notes

- Format clean across all 5 candidates. Brevity caps hold (candidates 14–17 words, rationales 17–22 words).
- Every rationale leads with the bite or the cheaper alternative: _"Atomic commits become the prize"_ / _"A shared package boundary is the real fix"_ / _"Tooling … often dissolves this"_ / _"CODEOWNERS and a service catalog usually outperform a repo merge"_ / _"Pipeline templating fixes most of this without a merge"_.
- The anti-monorepo bias is now structural: `3.`, `4.`, `5.` each name a cheaper non-monorepo fix in the rationale itself, then route the user away in `Leads to →`. The Recommendation line summarizes: _"only pays off for `1.` and `2.`; `3.`–`5.` have cheaper fixes."_
- Q1 of ~8 (was ~6 in earlier passes) — model judges this prompt warrants a deeper tree.
