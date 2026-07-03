---
title: Configure Attempts and Exponential Backoff for Transient Failures
impact: HIGH
tags: queues, bullmq, retries, backoff, resilience, job-options
---

## Configure Attempts and Exponential Backoff for Transient Failures

Background jobs routinely hit transient failures — a third-party API times out, the database is briefly unavailable, a rate limit is hit. With the default `attempts: 1`, the first failure is permanent and the work is silently lost. But naive immediate retries are also harmful: hammering a struggling downstream with back-to-back attempts makes the outage worse (a retry storm).

Set `attempts` to a small bound (3–5) and use **exponential backoff** (`type: 'exponential'`) so each retry waits progressively longer, giving the dependency time to recover. Configure it per job via `add()` options, or once for the whole queue via `defaultJobOptions`. Pair it with `removeOnComplete`/`removeOnFail` so Redis doesn't accumulate finished jobs forever, and keep the processor idempotent (see `idempotent-processors`) so retries are safe.

**Incorrect:**

```typescript
// Default attempts: 1 → one transient blip permanently loses the job.
await this.emailQueue.add('welcome', { userId });
```

**Correct:**

```typescript
// Per-job: bounded attempts + exponential backoff between the 4 retries (1s, 2s, 4s, 8s).
await this.emailQueue.add(
  'welcome',
  { userId },
  {
    attempts: 5,
    backoff: { type: 'exponential', delay: 1000 },
    removeOnComplete: 1000, // keep last 1000 completed
    removeOnFail: 5000,     // keep last 5000 failed for inspection
  },
);

// Or set defaults once for every job on the queue:
BullModule.registerQueue({
  name: 'email',
  defaultJobOptions: {
    attempts: 5,
    backoff: { type: 'exponential', delay: 1000 },
    removeOnComplete: 1000,
    removeOnFail: 5000,
  },
});
```
