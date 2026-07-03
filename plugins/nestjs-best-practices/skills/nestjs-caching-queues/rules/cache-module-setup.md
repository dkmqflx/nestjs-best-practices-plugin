---
title: Register CacheModule Globally — In-Memory for Dev, Redis for Prod
impact: HIGH
tags: caching, cache-module, redis, keyv, configuration, module-setup
---

## Register CacheModule Globally — In-Memory for Dev, Redis for Prod

The default `CacheModule.register()` stores entries in the process memory of a single instance. That is fine for local development, but in production it breaks the moment you run more than one replica: each instance has its own cache, so hit rates collapse and invalidation in one pod never reaches the others. Use a shared **Redis** store in production via `@keyv/redis`, and register `CacheModule` as a global module so you do not re-import it everywhere. Drive the store choice from configuration, not from a hardcoded environment check buried in `AppModule`.

Install: `npm i @nestjs/cache-manager cache-manager` (plus `@keyv/redis keyv cacheable` for Redis).

**Incorrect:**

```typescript
// In-memory only — not shared across replicas, and re-imported in every feature module.
import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [CacheModule.register()], // default in-memory, not global
})
export class UsersModule {}
```

**Correct:**

```typescript
import { Module } from '@nestjs/common';
import { CacheModule } from '@nestjs/cache-manager';
import { ConfigModule, ConfigService } from '@nestjs/config';
import KeyvRedis from '@keyv/redis';
import { Keyv } from 'keyv';
import { KeyvCacheableMemory } from 'cacheable';

@Module({
  imports: [
    CacheModule.registerAsync({
      isGlobal: true, // register once in AppModule; inject CACHE_MANAGER anywhere
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => {
        const redisUrl = config.get<string>('REDIS_URL');
        return {
          // TTL (milliseconds) lives on each store; see set-sensible-ttl.
          stores: redisUrl
            ? [new KeyvRedis(redisUrl)]
            : [new Keyv({ store: new KeyvCacheableMemory({ ttl: 30_000, lruSize: 5000 }) })],
        };
      },
    }),
  ],
})
export class AppModule {}
```
