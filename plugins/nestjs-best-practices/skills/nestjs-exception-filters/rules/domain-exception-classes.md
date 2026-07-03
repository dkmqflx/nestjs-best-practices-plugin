---
title: Model Domain Errors as Exception Classes
impact: MEDIUM
tags: custom-exception,http-exception,domain,error-code
---

## Model Domain Errors as Exception Classes

When the same business error is thrown from many places, repeating `throw new ConflictException('Email already in use')` scatters the message and status. Define a domain exception once by extending `HttpException` (or a built-in like `ConflictException`). It centralizes the status, message, and any stable machine-readable error code, and reads clearly at the call site.

**Incorrect:**

```typescript
// Repeated string + status everywhere this error occurs
if (existing) {
  throw new ConflictException('Email already in use');
}
// ...another file
if (taken) {
  throw new ConflictException('Email already in use'); // drifts over time
}
```

**Correct:**

```typescript
import { HttpException, HttpStatus } from '@nestjs/common';

export class EmailAlreadyInUseException extends HttpException {
  constructor(email: string) {
    super(
      { code: 'EMAIL_IN_USE', message: `Email ${email} is already in use` },
      HttpStatus.CONFLICT,
    );
  }
}

// Call site
if (existing) {
  throw new EmailAlreadyInUseException(dto.email);
}
```
