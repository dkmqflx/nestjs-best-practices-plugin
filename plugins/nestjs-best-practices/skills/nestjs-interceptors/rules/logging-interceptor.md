---
title: Measure handler duration with tap()
impact: MEDIUM
tags: logging,tap,rxjs,observability,latency
---

## Measure handler duration with tap()

Use `tap()` to run side effects (logging) without altering the response stream.
Capture a start timestamp before `next.handle()`, then log method, URL, and
elapsed time in the `tap()` callback — which fires after the handler completes.
Prefer Nest's `Logger` over `console.log`, and read request metadata from the
`ExecutionContext` so the interceptor works across HTTP, RPC, and WS contexts.

**Incorrect:**

```typescript
// map() mutates the response just to log — and timing is taken too late.
intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
  return next.handle().pipe(
    map((data) => {
      console.log('done');     // no method/url, no duration
      return data;
    }),
  );
}
```

**Correct:**

```typescript
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  Logger,
  NestInterceptor,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const { method, url } = context.switchToHttp().getRequest();
    const now = Date.now();

    return next.handle().pipe(
      tap(() => this.logger.log(`${method} ${url} ${Date.now() - now}ms`)),
    );
  }
}
```
