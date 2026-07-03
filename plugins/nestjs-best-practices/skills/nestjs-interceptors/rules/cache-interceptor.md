---
title: Cache GET responses with CacheInterceptor
impact: MEDIUM
tags: cache,CacheInterceptor,CacheModule,ttl,cache-manager,get
---

## Cache GET responses with CacheInterceptor

Don't hand-roll response caching. Install `@nestjs/cache-manager` and
`cache-manager`, register `CacheModule`, and apply the built-in
`CacheInterceptor`. It automatically caches and serves responses keyed by
request URL. Note two things: **only `GET` endpoints are cached**, and `ttl` is
in **milliseconds** (default `0` = never expires). Override per-route with
`@CacheKey()` / `@CacheTTL()`. Bind it globally via `APP_INTERCEPTOR` to cache
all GET routes.

**Incorrect:**

```typescript
// Manual, ad-hoc caching inside the controller — duplicated and error-prone.
private store = new Map<string, string[]>();

@Get()
findAll(): string[] {
  if (this.store.has('all')) return this.store.get('all')!;
  const data = this.heavyQuery();
  this.store.set('all', data); // no TTL, no eviction, leaks memory
  return data;
}
```

**Correct:**

```typescript
import { Module } from '@nestjs/common';
import { CacheModule, CacheInterceptor } from '@nestjs/cache-manager';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { AppController } from './app.controller';

@Module({
  imports: [CacheModule.register({ ttl: 5000 })], // ttl in milliseconds
  controllers: [AppController],
  providers: [{ provide: APP_INTERCEPTOR, useClass: CacheInterceptor }],
})
export class AppModule {}
```

```typescript
import { Controller, Get, UseInterceptors } from '@nestjs/common';
import { CacheInterceptor, CacheKey, CacheTTL } from '@nestjs/cache-manager';

@Controller()
@UseInterceptors(CacheInterceptor) // or rely on the global binding above
export class AppController {
  @Get()
  @CacheKey('all_cats')
  @CacheTTL(20000)
  findAll(): string[] {
    return [];
  }
}
```
