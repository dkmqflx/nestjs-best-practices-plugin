---
title: Register ValidationPipe Globally
impact: CRITICAL
tags: validation,validationpipe,main,bootstrap,global
---

## Register ValidationPipe Globally

Binding `ValidationPipe` once in `main.ts` guarantees every endpoint validates its input. Wiring it per-handler is easy to forget and leaves routes unprotected.

**Incorrect:**

```typescript
// Each controller method opts in by hand — easy to miss
@Post()
create(@Body(new ValidationPipe()) dto: CreateUserDto) {}
```

**Correct:**

```typescript
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(
    new ValidationPipe({ whitelist: true, transform: true }),
  );
  await app.listen(3000);
}
bootstrap();
```
