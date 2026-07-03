---
title: Avoid request-scoped providers
impact: HIGH
tags: performance,scope,di,providers
---

## Avoid request-scoped providers

`Scope.REQUEST` forces NestJS to instantiate the provider — and its entire injection chain up to the controller — on every request, adding allocation and resolution overhead that kills throughput. Keep providers as the default singleton scope. When you only need per-request data (like the user or a correlation ID), inject the request or use an AsyncLocalStorage-based context service instead of bumping the scope.

**Incorrect:**

```typescript
@Injectable({ scope: Scope.REQUEST }) // re-created per request, taints the chain
export class AuditService {
  constructor(@Inject(REQUEST) private req: Request) {}
  log(action: string) { /* uses this.req.user */ }
}
```

**Correct:**

```typescript
@Injectable() // singleton
export class AuditService {
  // pass per-request data in, or read from an ALS-backed ContextService
  log(action: string, userId: string) { /* ... */ }
}
```
