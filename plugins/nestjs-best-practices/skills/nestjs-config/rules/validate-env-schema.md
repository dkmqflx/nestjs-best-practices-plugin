---
title: Validate environment at boot so misconfig fails fast
impact: CRITICAL
tags: validation, joi, class-validator, fail-fast
---

## Validate environment at boot so misconfig fails fast

A missing or malformed environment variable should crash the app at startup, not surface
as an `undefined` deep inside a request handler in production. `@nestjs/config` validates
env at bootstrap when you pass either a `validationSchema` (Joi) or a `validate` function
(class-validator). Either way, the process refuses to start on invalid config.

**Incorrect:**

```typescript
// No validation — a typo'd or missing PORT silently becomes undefined at runtime
ConfigModule.forRoot({ isGlobal: true });

const port = config.get('PORT'); // undefined → app listens on a random/wrong port
```

**Correct:**

```typescript
// Joi schema validated at boot; defaults applied, unknowns rejected
import * as Joi from 'joi';

ConfigModule.forRoot({
  isGlobal: true,
  validationSchema: Joi.object({
    NODE_ENV: Joi.string()
      .valid('development', 'production', 'test')
      .default('development'),
    PORT: Joi.number().port().default(3000),
    DATABASE_URL: Joi.string().uri().required(),
  }),
  validationOptions: {
    allowUnknown: true, // allow other env vars (e.g. system ones)
    abortEarly: false,  // report all errors, not just the first
  },
});
```
