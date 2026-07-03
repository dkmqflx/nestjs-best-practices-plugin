---
title: Restructure to Avoid Circular Dependencies; forwardRef Last
impact: HIGH
tags: circular-dependency, forwardRef, architecture
---

## Restructure to Avoid Circular Dependencies; forwardRef Last

A circular dependency between two providers or modules usually signals a design problem. Prefer to break the cycle: extract the shared logic into a third provider/module, or invert the dependency so only one side knows the other. Reach for `forwardRef()` (or `ModuleRef` for runtime resolution) only when restructuring is genuinely impractical.

**Incorrect:**

```typescript
// A <-> B mutually inject each other directly
@Injectable()
class UsersService { constructor(private orders: OrdersService) {} }
@Injectable()
class OrdersService { constructor(private users: UsersService) {} }
```

**Correct:**

```typescript
// Extract shared logic so the cycle disappears
@Injectable()
class MembershipService { /* logic both needed */ }

@Injectable()
class UsersService { constructor(private membership: MembershipService) {} }
@Injectable()
class OrdersService { constructor(private membership: MembershipService) {} }

// Last resort only, when the cycle cannot be removed:
@Injectable()
class OrdersService {
  constructor(
    @Inject(forwardRef(() => UsersService))
    private readonly users: UsersService,
  ) {}
}
// Modules: imports: [forwardRef(() => UsersModule)]
```

Reference: https://docs.nestjs.com/fundamentals/circular-dependency
