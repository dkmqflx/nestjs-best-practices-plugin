---
title: Strip or Reject Unknown Properties
impact: HIGH
tags: validation,whitelist,security,forbidNonWhitelisted
---

## Strip or Reject Unknown Properties

Without `whitelist`, clients can send extra properties that bypass validation and may reach your persistence layer (mass-assignment risk). `whitelist: true` strips undecorated props; `forbidNonWhitelisted: true` rejects them with a 400.

**Incorrect:**

```typescript
// Extra fields like { isAdmin: true } pass straight through
app.useGlobalPipes(new ValidationPipe());
```

**Correct:**

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
  }),
);
```
