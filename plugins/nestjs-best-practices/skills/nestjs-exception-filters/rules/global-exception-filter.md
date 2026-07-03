---
title: Register a Global Exception Filter via APP_FILTER
impact: HIGH
tags: app-filter,global,exception-filter,catch,di
---

## Register a Global Exception Filter via APP_FILTER

To format every error consistently, register one global filter. Prefer the `APP_FILTER` provider over `app.useGlobalFilters(new MyFilter())`: `APP_FILTER` is module-scoped, so the filter participates in dependency injection (it can inject a logger, config, `HttpAdapterHost`, etc.). The `useGlobalFilters` instance form is created outside the DI container and cannot inject providers.

**Incorrect:**

```typescript
// main.ts — instance is outside DI, cannot inject dependencies
app.useGlobalFilters(new AllExceptionsFilter(/* no way to inject Logger */));
```

**Correct:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';
import { AllExceptionsFilter } from './all-exceptions.filter';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: AllExceptionsFilter, // resolved by DI, deps injectable
    },
  ],
})
export class AppModule {}
```
