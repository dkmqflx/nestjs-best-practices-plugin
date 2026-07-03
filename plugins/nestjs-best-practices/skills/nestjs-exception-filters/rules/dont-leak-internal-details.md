---
title: Never Leak Internal Details to Clients
impact: CRITICAL
tags: security,logging,stack-trace,information-disclosure
---

## Never Leak Internal Details to Clients

For unexpected (non-`HttpException`) errors, the raw message often contains internals — a stack trace, a SQL query, a connection string, or a secret. Returning those to the client is an information-disclosure vulnerability. In your catch-all filter, log the full error server-side (with a logger, not `console.log`) and return a generic, opaque message for 5xx errors. Only the controlled messages from `HttpException` subclasses should reach the client.

**Incorrect:**

```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const res = host.switchToHttp().getResponse();
    // Leaks stack trace / SQL / secrets straight to the client
    res.status(500).json({ message: (exception as Error).stack });
  }
}
```

**Correct:**

```typescript
import { Catch, ArgumentsHost, ExceptionFilter, HttpException, HttpStatus, Logger } from '@nestjs/common';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);

  catch(exception: unknown, host: ArgumentsHost) {
    const res = host.switchToHttp().getResponse();
    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    // Full detail stays on the server
    this.logger.error(exception instanceof Error ? exception.stack : String(exception));

    const message =
      exception instanceof HttpException
        ? exception.getResponse()
        : 'Internal server error'; // opaque for 5xx

    res.status(status).json({ statusCode: status, message });
  }
}
```
