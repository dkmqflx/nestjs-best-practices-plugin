---
title: Enable graceful shutdown hooks
impact: HIGH
tags: lifecycle,shutdown,kubernetes,reliability
---

## Enable graceful shutdown hooks

Without shutdown hooks, a SIGTERM kills the process mid-request, dropping in-flight work and leaking DB/queue connections. Call `app.enableShutdownHooks()` and implement `OnModuleDestroy` (or `beforeApplicationShutdown`) to stop accepting traffic, finish in-flight requests, and close resources. This is what lets orchestrators roll pods safely.

**Incorrect:**

```typescript
const app = await NestFactory.create(AppModule);
await app.listen(3000);
// SIGTERM -> hard exit, open connections leak, requests dropped
```

**Correct:**

```typescript
// main.ts
const app = await NestFactory.create(AppModule);
app.enableShutdownHooks();
await app.listen(3000);

// database.service.ts
@Injectable()
export class DatabaseService implements OnModuleDestroy {
  async onModuleDestroy() {
    await this.pool.end(); // drain & close connections cleanly
  }
}
```
