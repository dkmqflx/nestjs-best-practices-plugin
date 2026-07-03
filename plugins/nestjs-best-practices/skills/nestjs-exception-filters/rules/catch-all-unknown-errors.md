---
title: Catch Unknown Errors and Map Them to 500
impact: HIGH
tags: catch-all,exception-filter,http-adapter,500,unknown-error
---

## Catch Unknown Errors and Map Them to 500

A filter declared with `@Catch(HttpException)` only catches `HttpException` subclasses — a raw `TypeError`, a database driver error, or a thrown string slips past it and falls back to the default handler. Add a catch-all filter (`@Catch()` with no argument) that handles everything: it normalizes `HttpException` statuses and maps any non-`HttpException` to `500 INTERNAL_SERVER_ERROR`. Use `HttpAdapterHost` so the filter stays framework-agnostic (works under Express or Fastify).

**Incorrect:**

```typescript
@Catch(HttpException) // a TypeError from buggy code is NOT caught here
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    ctx.getResponse().status(exception.getStatus()).json(exception.getResponse());
  }
}
```

**Correct:**

```typescript
import {
  Catch,
  ArgumentsHost,
  ExceptionFilter,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { HttpAdapterHost } from '@nestjs/core';

@Catch() // no argument -> catches EVERYTHING
export class AllExceptionsFilter implements ExceptionFilter {
  constructor(private readonly httpAdapterHost: HttpAdapterHost) {}

  catch(exception: unknown, host: ArgumentsHost): void {
    const { httpAdapter } = this.httpAdapterHost;
    const ctx = host.switchToHttp();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    httpAdapter.reply(ctx.getResponse(), { statusCode: status }, status);
  }
}
```
