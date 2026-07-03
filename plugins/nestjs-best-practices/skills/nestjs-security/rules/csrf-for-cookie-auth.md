---
title: Add CSRF Protection for Cookie-Based Sessions
impact: HIGH
tags: csrf,cookies,sessions,double-submit,express,fastify
---

## Add CSRF Protection for Cookie-Based Sessions

When auth relies on a cookie the browser sends automatically, a malicious site can forge state-changing requests on the user's behalf (CSRF). Token-based auth carried in an `Authorization` header is not vulnerable and does not need this. For cookie/session auth, add CSRF middleware: Express uses `csrf-csrf` (double-submit), which requires `cookie-parser` or session middleware initialized first; Fastify uses `@fastify/csrf-protection`, which needs a storage plugin registered first.

**Incorrect:**

```typescript
// Cookie-session auth with no CSRF defense
app.use(cookieParser());
app.use(session({ secret: process.env.SESSION_SECRET }));
// state-changing POST/PUT/DELETE routes are forgeable cross-site
```

**Correct:**

```typescript
// main.ts (Express) — cookie-parser MUST come first
import * as cookieParser from 'cookie-parser';
import { doubleCsrf } from 'csrf-csrf';

app.use(cookieParser());

const { doubleCsrfProtection } = doubleCsrf({
  getSecret: () => process.env.CSRF_SECRET,
});
app.use(doubleCsrfProtection);

// main.ts (Fastify) — register a storage/session plugin first, then:
// import fastifyCsrf from '@fastify/csrf-protection';
// await app.register(fastifyCsrf);
```
