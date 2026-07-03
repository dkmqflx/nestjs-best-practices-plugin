---
title: Prefer Constructor Injection over Property Injection
impact: MEDIUM
tags: providers, dependency-injection, testability
---

## Prefer Constructor Injection over Property Injection

Declare dependencies as `private readonly` constructor parameters. Constructor injection makes dependencies explicit, guarantees they exist before any method runs, and keeps classes trivially testable (pass mocks to the constructor). Property injection with `@Inject()` exists mainly for cases where the base class cannot expose a constructor; it hides dependencies and complicates instantiation in tests.

**Incorrect:**

```typescript
@Injectable()
export class OrdersService {
  @Inject(PaymentService)
  private readonly payment: PaymentService; // hidden dependency
}
```

**Correct:**

```typescript
@Injectable()
export class OrdersService {
  constructor(private readonly payment: PaymentService) {}
}
```

Reference: https://docs.nestjs.com/providers
