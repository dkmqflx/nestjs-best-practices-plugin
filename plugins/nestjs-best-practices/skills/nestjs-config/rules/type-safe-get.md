---
title: Type ConfigService.get and use infer to avoid undefined surprises
impact: MEDIUM
tags: type-safety, generics, infer, configservice
---

## Type ConfigService.get and use infer to avoid undefined surprises

By default `ConfigService.get()` returns a loosely typed value, so a misspelled key or
missing var silently flows through as `undefined`. Parameterize `ConfigService` with an
interface of your env shape and pass `{ infer: true }` so return types are inferred and
keys are checked. Use the strict second generic (`true`) to make `get()` non-nullable for
keys known to exist.

**Incorrect:**

```typescript
// Untyped: no key checking, return type is unknown/any-ish
constructor(private readonly config: ConfigService) {}

start() {
  const port = this.config.get('PORT'); // could be undefined, no type help
  app.listen(port);
}
```

**Correct:**

```typescript
interface EnvironmentVariables {
  PORT: number;
  NODE_ENV: 'development' | 'production' | 'test';
}

// Second generic `true` (WasValidated) → get() returns non-nullable types
constructor(
  private readonly config: ConfigService<EnvironmentVariables, true>,
) {}

start() {
  const port = this.config.get('PORT', { infer: true }); // typed as number
  const env = this.config.get('NODE_ENV', { infer: true }); // typed union
  app.listen(port);
}
```
