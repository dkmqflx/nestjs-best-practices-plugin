---
title: Rate Limit Requests with @nestjs/throttler
impact: HIGH
tags: rate-limiting,throttler,dos,brute-force,guard
---

## Rate Limit Requests with @nestjs/throttler

Unbounded endpoints invite brute-force attacks on auth, credential stuffing, and DoS. `@nestjs/throttler` caps requests per client over a time window. Register `ThrottlerModule.forRoot` with a `throttlers` array (v6+ shape) and wire `ThrottlerGuard` as a global `APP_GUARD` so every route is protected by default. Use `@Throttle()` to tighten sensitive routes (e.g. login) and `@SkipThrottle()` to exempt webhooks or health checks.

**Incorrect:**

```typescript
// No throttling — /auth/login can be hammered indefinitely
@Module({
  imports: [],
  controllers: [AuthController],
})
export class AppModule {}
```

**Correct:**

```typescript
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { ThrottlerModule, ThrottlerGuard } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot({
      throttlers: [{ ttl: 60000, limit: 10 }], // 10 req / 60s
    }),
  ],
  providers: [{ provide: APP_GUARD, useClass: ThrottlerGuard }],
})
export class AppModule {}

// Tighten a sensitive route:
import { Throttle } from '@nestjs/throttler';

@Throttle({ default: { limit: 3, ttl: 60000 } })
@Post('login')
login() { /* ... */ }
```
