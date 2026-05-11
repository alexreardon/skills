# 04. Background jobs in Node.js

**Prompt:**
> "I need to add background job processing to my Node.js app. What should I use?"

---

## `/grill-me` output

What's the primary workload shape for these background jobs? My recommendation: **deferred I/O-bound tasks** (sending emails, webhooks, third-party API calls, notifications) — this is the most common case and shapes everything downstream (queue choice, concurrency model, retry semantics). Other branches to consider: CPU-heavy work (image/video processing, PDF generation), scheduled/cron-style recurring jobs, long-running workflows with multiple steps, or fan-out/fan-in batch processing.

Which is it?

---

## `/cook-me` output

**`Q1 of ~6.`**
> What's driving the need for background jobs — what work are you trying to move off the request path?

**`Candidates`** _(↓ most to least promising)_

`1.` **Slow user-triggered work**<br>
Emails, PDFs, image processing, third-party API calls during requests.<br>
_Latency dominates the decision; you need durability and retries but can tolerate seconds-to-minutes delay._<br>
`Leads to →` Redis-backed queue (BullMQ)

`2.` **Scheduled / recurring work**<br>
Nightly reports, cleanups, polling, cache warming on a cron.<br>
_Scheduling semantics dominate; missed-run behavior and overlap prevention matter more than throughput._<br>
`Leads to →` BullMQ repeatable jobs or Agenda

`3.` **Fan-out / event-driven pipelines**<br>
One event triggers many downstream steps across services or workers.<br>
_Topology becomes the hard problem; ordering, idempotency, and DLQs eclipse the "queue library" choice._<br>
`Leads to →` SQS/SNS, Kafka, or Temporal

`4.` **Long-running workflows**<br>
Multi-step processes spanning minutes-to-days with human approvals or external waits.<br>
_State persistence across restarts is the bite; ad-hoc queues will drift into a half-built workflow engine._<br>
`Leads to →` Temporal or Inngest

**Recommendation:** Most Node apps land on `1.` — pinning which bucket dominates changes the answer materially, so worth confirming before ranking tools.

_**Next step:** Pick the bucket that fits, or describe the actual workload._

---

## Notes

- Format clean across all 4 candidates. Brevity caps hold (candidates 10–13 words, rationales 13–18 words).
- Every rationale leads with the bite — `4.` even uses the word _"bite"_ verbatim (_"State persistence across restarts is the bite"_), suggesting the model has internalized the rule's vocabulary.
- Standout rationale: `3.`'s _"Topology becomes the hard problem; ordering, idempotency, and DLQs eclipse the 'queue library' choice"_ — the same _"the framing is wrong"_ move that drove this prompt's previous-pass _"log not a job table"_ moment, now cast as a topology insight rather than a tool comparison.
