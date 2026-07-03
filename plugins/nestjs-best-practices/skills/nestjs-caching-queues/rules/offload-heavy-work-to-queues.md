---
title: Offload Slow or External Work to a Queue, Not the Request Path
impact: CRITICAL
tags: queues, bullmq, background-jobs, latency, producers, event-loop
---

## Offload Slow or External Work to a Queue, Not the Request Path

Doing slow or unreliable work synchronously inside a request handler hurts in two ways: the user waits for it (sending an email or transcoding a video can take seconds), and a failure in that work fails the whole HTTP request even though the primary action succeeded. CPU-bound work (image/video processing, PDF generation) additionally blocks the Node.js event loop, stalling every other request on that instance.

Move such work into a BullMQ queue: the handler does the fast, must-succeed part (e.g., create the order), enqueues a job for the rest, and returns immediately. A consumer processes the job asynchronously with its own retry semantics. Good candidates: email/SMS/push notifications, image/video/audio processing, third-party API calls, report generation, webhooks. Inject the queue with `@InjectQueue` and add jobs with `queue.add(name, data, options)`.

Install: `npm i @nestjs/bullmq bullmq`, then register the queue with `BullModule.registerQueue({ name: 'email' })`.

**Incorrect:**

```typescript
@Post()
async register(@Body() dto: RegisterDto) {
  const user = await this.users.create(dto);
  // Blocks the response on a slow external call; if SMTP is down, registration fails.
  await this.mailer.sendWelcomeEmail(user.email);
  return user;
}
```

**Correct:**

```typescript
import { InjectQueue } from '@nestjs/bullmq';
import { Queue } from 'bullmq';

@Injectable()
export class UsersService {
  constructor(@InjectQueue('email') private emailQueue: Queue) {}

  async register(dto: RegisterDto) {
    const user = await this.users.create(dto);
    // Fast: enqueue and return. The email is delivered by a consumer, with retries.
    await this.emailQueue.add('welcome', { userId: user.id, email: user.email });
    return user;
  }
}
```
