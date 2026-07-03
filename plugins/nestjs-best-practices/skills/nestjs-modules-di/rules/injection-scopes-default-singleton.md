---
title: Prefer the Default Singleton Scope
impact: HIGH
tags: injection-scopes, performance, providers
---

## Prefer the Default Singleton Scope

By default providers are singletons — one shared instance for the whole app, which is the most performant choice for stateless services. `Scope.REQUEST` creates a new instance per request and **bubbles up**: every provider and controller in the injection chain that depends on it also becomes request-scoped, losing singleton optimization. Reserve REQUEST scope for genuine per-request state, and pass request data as method arguments instead where possible.

**Incorrect:**

```typescript
// Forces a new instance per request for a stateless service,
// and makes every consumer request-scoped too
@Injectable({ scope: Scope.REQUEST })
export class PricingService {
  calculate(order: Order) { /* no request state needed */ }
}
```

**Correct:**

```typescript
@Injectable() // default singleton
export class PricingService {
  calculate(order: Order) { /* ... */ }
}

// If you truly need the request, inject it only where required:
@Injectable({ scope: Scope.REQUEST })
export class AuditContext {
  constructor(@Inject(REQUEST) private readonly req: Request) {}
}
```

Reference: https://docs.nestjs.com/fundamentals/injection-scopes
