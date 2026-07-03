---
title: Make Job Processors Idempotent — Tolerate Retries and Duplicates
impact: CRITICAL
tags: queues, bullmq, processor, idempotency, retries, worker-host
---

## Make Job Processors Idempotent — Tolerate Retries and Duplicates

BullMQ delivers jobs **at least once**. A job can run more than once: it is retried after a failure, and it can be re-delivered if the worker crashes after doing the work but before acking, or after a stall is detected. A processor that performs a non-idempotent side effect (charge a card, send an email, increment a counter) will therefore eventually double-execute. The fix is not "avoid retries" — retries are essential — it is to make the side effect safe to repeat.

Make processors idempotent: derive a stable idempotency key from the job (use a deterministic `jobId` or a domain key in `job.data`), and either check-before-acting or use an upsert / unique constraint so a second run is a no-op. A BullMQ consumer extends `WorkerHost` and implements `process(job)`; branch on `job.name` for named jobs.

**Incorrect:**

```typescript
import { Processor, WorkerHost } from '@nestjs/bullmq';
import { Job } from 'bullmq';

@Processor('payments')
export class PaymentsConsumer extends WorkerHost {
  async process(job: Job<{ orderId: string; amount: number }>) {
    // Runs again on every retry/redelivery → the customer is charged twice.
    await this.stripe.charge(job.data.amount);
    await this.orders.markPaid(job.data.orderId);
  }
}
```

**Correct:**

```typescript
import { Processor, WorkerHost } from '@nestjs/bullmq';
import { Job } from 'bullmq';

@Processor('payments')
export class PaymentsConsumer extends WorkerHost {
  async process(job: Job<{ orderId: string; amount: number }>) {
    const { orderId, amount } = job.data;

    // Idempotency guard: a re-run is a no-op once the order is already paid.
    if (await this.orders.isPaid(orderId)) return;

    // Pass a stable key so the provider dedupes a repeated charge on its side too.
    await this.stripe.charge(amount, { idempotencyKey: `order:${orderId}` });
    await this.orders.markPaid(orderId);
  }
}
```
