---
title: Register ConfigModule globally, once, at the root
impact: HIGH
tags: configmodule, forroot, isglobal, module
---

## Register ConfigModule globally, once, at the root

Call `ConfigModule.forRoot({ isGlobal: true })` a single time in the root `AppModule`.
With `isGlobal: true`, `ConfigService` is available everywhere without re-importing
`ConfigModule` into each feature module. Re-importing per module reloads/re-parses env
files and invites inconsistent options across modules.

**Incorrect:**

```typescript
// Every feature module re-imports ConfigModule with its own options
@Module({
  imports: [ConfigModule.forRoot()], // repeated in users.module.ts, orders.module.ts, ...
})
export class UsersModule {}
```

**Correct:**

```typescript
// app.module.ts — registered once, globally
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
  ],
})
export class AppModule {}

// Any feature module can now inject ConfigService without importing ConfigModule
@Module({ providers: [UsersService] })
export class UsersModule {}
```
