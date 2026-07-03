---
title: Always Set a TTL (in Milliseconds) — Never Cache Unbounded
impact: HIGH
tags: caching, ttl, expiration, staleness, milliseconds
---

## Always Set a TTL (in Milliseconds) — Never Cache Unbounded

In `@nestjs/cache-manager`, the default `ttl` is `0`, which means **never expire**. A cache entry that never expires becomes a permanent source of stale data and a slow memory leak — values that changed in the database are served forever from the cache. Always pick an explicit TTL that matches how stale the data is allowed to be: short for volatile data (a few seconds), longer for near-static reference data (minutes to hours).

The unit gotcha: since `@nestjs/cache-manager` v2 / `cache-manager` v5+, **all TTLs are in milliseconds**, not seconds. `ttl: 60` means 60 ms, not one minute — a classic source of caches that appear to "not work." Use numeric separators (`60_000`) to keep intent obvious. The per-`set` TTL is the third argument and overrides the module default.

**Incorrect:**

```typescript
// ttl: 60 is 60 MILLISECONDS — expires almost immediately, not "1 minute".
await this.cacheManager.set('user:42', user, 60);

// No TTL at all → never expires → permanently stale after the user changes.
await this.cacheManager.set('user:42', user);
```

**Correct:**

```typescript
// Explicit, in milliseconds, sized to acceptable staleness.
const FIVE_MINUTES = 5 * 60_000;
await this.cacheManager.set('user:42', user, FIVE_MINUTES);

// Module-level default (also milliseconds) for entries set without an explicit TTL.
CacheModule.register({ isGlobal: true, ttl: 30_000 }); // 30 seconds
```
