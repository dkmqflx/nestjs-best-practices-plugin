---
title: Reserve @Global() for Truly Cross-Cutting Modules
impact: MEDIUM
tags: modules, global, encapsulation, architecture
---

## Reserve @Global() for Truly Cross-Cutting Modules

`@Global()` registers a module's exports once and makes them injectable everywhere without an explicit `imports`. That convenience defeats NestJS's encapsulation and hides dependencies, so limit it to genuinely app-wide concerns like configuration or logging. For everything else, import the module explicitly where it is used.

**Incorrect:**

```typescript
// A feature module made global just to avoid typing imports
@Global()
@Module({ providers: [OrdersService], exports: [OrdersService] })
export class OrdersModule {}
```

**Correct:**

```typescript
// Cross-cutting concern: acceptable as global
@Global()
@Module({ providers: [ConfigService], exports: [ConfigService] })
export class ConfigModule {}

// Feature module: import explicitly where needed
@Module({ providers: [OrdersService], exports: [OrdersService] })
export class OrdersModule {}
```

Make a module global at most once; dynamic modules can set `global: true` in the returned `DynamicModule`. Reference: https://docs.nestjs.com/modules
