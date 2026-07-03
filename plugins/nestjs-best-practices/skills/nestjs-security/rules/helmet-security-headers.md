---
title: Apply Helmet Security Headers
impact: HIGH
tags: headers,helmet,express,fastify,xss,clickjacking
---

## Apply Helmet Security Headers

By default NestJS sends no protective HTTP headers, leaving the app exposed to clickjacking, MIME sniffing, and content-injection attacks. Helmet sets sensible security headers (CSP, X-Content-Type-Options, etc.) in one call. It must be registered **before** any other middleware so the headers apply to every route. The package differs by platform: Express uses `helmet`, Fastify uses `@fastify/helmet`.

**Incorrect:**

```typescript
// main.ts — no security headers at all
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

**Correct:**

```typescript
// main.ts (Express — default adapter)
import { NestFactory } from '@nestjs/core';
import helmet from 'helmet';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(helmet()); // register first, before other middleware
  await app.listen(3000);
}
bootstrap();

// main.ts (Fastify) — register as a plugin instead
// import helmet from '@fastify/helmet';
// await app.register(helmet);
```
