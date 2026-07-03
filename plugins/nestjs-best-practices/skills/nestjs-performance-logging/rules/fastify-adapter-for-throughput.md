---
title: Prefer FastifyAdapter for throughput
impact: HIGH
tags: performance,fastify,http,adapter
---

## Prefer FastifyAdapter for throughput

Fastify handles roughly 2x the requests per second of Express thanks to its low-overhead routing and schema-based serialization. For high-throughput HTTP services, the Fastify adapter is a near-drop-in win — just account for the different underlying request/response objects in any low-level middleware.

**Incorrect:**

```typescript
// Default Express adapter — fine, but leaves throughput on the table
const app = await NestFactory.create(AppModule);
await app.listen(process.env.PORT ?? 3000);
```

**Correct:**

```typescript
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter(),
  );
  await app.listen(process.env.PORT ?? 3000, '0.0.0.0');
}
bootstrap();
```
