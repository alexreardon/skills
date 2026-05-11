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
> What's driving the need — offloading slow work from request handlers, scheduling future/recurring tasks, or coordinating multi-step workflows?

**`Candidates`** _(↓ most to least promising)_

`1.` **Offload slow work**<br>
Emails, image processing, webhooks — anything making HTTP responses sluggish.<br>
_Latency is the bite; if a job dies mid-run, the user already got a 200 and won't retry._<br>
`Leads to →` BullMQ, pg-boss (+ more)

`2.` **Scheduled / recurring jobs**<br>
Nightly reports, cleanup, polling external APIs on a cadence.<br>
_Missed runs silently compound; you need cron semantics plus catch-up and overlap guards, not just a queue._<br>
`Leads to →` BullMQ repeatable, Agenda

`3.` **Durable workflows**<br>
Multi-step pipelines with retries, branching, human-in-the-loop, or long waits.<br>
_State across steps becomes the hard part; ad-hoc queues drift into bespoke orchestrators you'll regret owning._<br>
`Leads to →` Temporal, Inngest

`4.` **Fan-out / event processing**<br>
One event triggers many downstream consumers across services.<br>
_Ordering and at-least-once delivery dominate; you're really picking a broker, not a job runner._<br>
`Leads to →` Kafka, NATS, SQS+SNS

**Recommendation:** `1.` — most Node apps reach for background jobs first to unblock request handlers; pinning this changes the entire shortlist.

_**Next step:** Pick the closest match, or describe the workload if it spans more than one._

---

## Notes

- Format clean across all 4 candidates. Brevity caps hold.
- Every rationale leads with the bite — `1.` even uses the word _"bite"_ verbatim (_"Latency is the bite"_), and the others follow suit (_"Missed runs silently compound"_ / _"State across steps becomes the hard part"_ / _"Ordering and at-least-once delivery dominate"_).
- New `Leads to →` rule heavily exercised: every candidate lists multiple tools. `1.` uses `(+ more)` to signal there's more beyond `BullMQ, pg-boss`; `4.` lists three brokers because the broker pick depends on existing infra, not job-shape.
- Standout rationale: `4.`'s _"You're really picking a broker, not a job runner"_ — the same _"the framing is wrong"_ move that previously surfaced as _"log not a job table"_, now cast as a broker-vs-runner reframe.
