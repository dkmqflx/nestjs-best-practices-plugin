---
title: Allowlist Explicit CORS Origins
impact: HIGH
tags: cors,origin,credentials,csrf,browser
---

## Allowlist Explicit CORS Origins

A wildcard origin combined with `credentials: true` is the worst CORS misconfiguration: browsers will let any site send authenticated cross-origin requests with the user's cookies. The CORS spec forbids `*` with credentials, but reflecting the request origin (`origin: true`) is the same hole in practice. Pass an explicit allowlist of trusted origins instead, and only enable `credentials` when you actually use cookie auth.

**Incorrect:**

```typescript
// Any website can make credentialed requests to this API
app.enableCors({
  origin: true, // reflects every request origin
  credentials: true,
});
```

**Correct:**

```typescript
// main.ts
const allowedOrigins = (process.env.CORS_ORIGINS ?? '').split(',').filter(Boolean);

app.enableCors({
  origin: allowedOrigins, // e.g. ['https://app.example.com']
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
});
```
