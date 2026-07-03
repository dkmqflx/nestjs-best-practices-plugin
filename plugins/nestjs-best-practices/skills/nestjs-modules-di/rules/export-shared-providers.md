---
title: Export Only the Providers You Intend to Share
impact: HIGH
tags: modules, providers, exports, encapsulation
---

## Export Only the Providers You Intend to Share

A provider is only injectable in another module if the owning module lists it in `exports` and the consumer `imports` that module. Importing a module does not expose its private providers. Export deliberately so each module's public surface stays small and intentional.

**Incorrect:**

```typescript
// UsersService not exported, yet OrdersModule tries to inject it
@Module({ providers: [UsersService] })
export class UsersModule {}

@Module({ imports: [UsersModule], providers: [OrdersService] })
export class OrdersModule {} // OrdersService cannot resolve UsersService
```

**Correct:**

```typescript
@Module({
  providers: [UsersService],
  exports: [UsersService], // public surface
})
export class UsersModule {}

@Module({
  imports: [UsersModule], // now UsersService is injectable here
  providers: [OrdersService],
})
export class OrdersModule {}
```

A module may re-export modules it imports (`exports: [UsersModule]`) to compose shared modules.
