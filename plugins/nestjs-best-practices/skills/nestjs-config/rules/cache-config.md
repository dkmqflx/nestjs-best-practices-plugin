---
title: Enable cache:true and mind env-file load order
impact: LOW
tags: cache, performance, envfilepath, load-order
---

## Enable cache:true and mind env-file load order

Set `cache: true` so `ConfigService` reads from `process.env` once and serves subsequent
`get()` calls from memory — useful when config is read on hot paths. Also be deliberate
about `envFilePath`: when an array is given, the first file that defines a key wins, so
list environment-specific overrides before shared defaults.

**Incorrect:**

```typescript
// No caching; defaults file listed before the override, so overrides never apply
ConfigModule.forRoot({
  isGlobal: true,
  envFilePath: ['.env', '.env.development'], // .env wins → .env.development ignored
});
```

**Correct:**

```typescript
// Cache enabled; most-specific file first so its keys take precedence
ConfigModule.forRoot({
  isGlobal: true,
  cache: true,
  envFilePath: ['.env.development.local', '.env.development', '.env'],
});
```
