---
title: Bind globally with APP_INTERCEPTOR, scope with @UseInterceptors
impact: MEDIUM
tags: binding,APP_INTERCEPTOR,UseInterceptors,global,dependency-injection
---

## Bind globally with APP_INTERCEPTOR, scope with @UseInterceptors

Choose the narrowest scope that fits. For one route or controller, use
`@UseInterceptors(MyInterceptor)` — Nest instantiates it through the DI
container, so the class can have injected dependencies. For app-wide behavior,
prefer registering a provider with the `APP_INTERCEPTOR` token over
`app.useGlobalInterceptors(new MyInterceptor())`: the token form is created
inside a module, so it **can inject dependencies**, whereas passing a manually
`new`-ed instance to `useGlobalInterceptors` cannot.

**Incorrect:**

```typescript
// Global via a hand-constructed instance — DI is dead, deps must be passed by hand.
const app = await NestFactory.create(AppModule);
app.useGlobalInterceptors(new LoggingInterceptor(/* can't inject anything */));
```

**Correct:**

```typescript
// Global + DI-friendly: registered as a provider with the APP_INTERCEPTOR token.
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { LoggingInterceptor } from './logging.interceptor';

@Module({
  providers: [
    { provide: APP_INTERCEPTOR, useClass: LoggingInterceptor },
  ],
})
export class AppModule {}
```

```typescript
// Scoped: controller- or route-level, also resolved through DI.
@UseInterceptors(LoggingInterceptor)
export class CatsController {}
```
