---
title: One Module per Feature, with Encapsulated Providers
impact: HIGH
tags: modules, architecture, encapsulation
---

## One Module per Feature, with Encapsulated Providers

Each feature or domain should own a module that groups its controllers and providers. Modules are the unit of encapsulation in NestJS — providers declared in one module are private to it unless exported, which keeps boundaries explicit and dependency graphs navigable.

**Incorrect:**

```typescript
// AppModule becomes a dumping ground for every provider
@Module({
  controllers: [UsersController, OrdersController, AuthController],
  providers: [UsersService, OrdersService, AuthService, MailService],
})
export class AppModule {}
```

**Correct:**

```typescript
@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

@Module({
  imports: [UsersModule, OrdersModule, AuthModule],
})
export class AppModule {}
```

Reference: https://docs.nestjs.com/modules
