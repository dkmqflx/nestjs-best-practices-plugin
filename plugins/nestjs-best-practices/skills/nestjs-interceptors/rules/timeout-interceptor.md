---
title: Fail slow requests with timeout()
impact: HIGH
tags: timeout,rxjs,catchError,RequestTimeoutException,resilience
---

## Fail slow requests with timeout()

Pipe the RxJS `timeout()` operator onto `next.handle()` to bound how long a
request may run. When it fires, RxJS emits a `TimeoutError`; catch it with
`catchError()` and rethrow a Nest `RequestTimeoutException` (HTTP 408) so the
client gets a clean error instead of a hung connection. Re-throw any other error
unchanged so you don't swallow real failures.

**Incorrect:**

```typescript
// timeout() alone surfaces a raw RxJS TimeoutError → 500 Internal Server Error.
intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
  return next.handle().pipe(timeout(5000));
}
```

**Correct:**

```typescript
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
  RequestTimeoutException,
} from '@nestjs/common';
import { Observable, throwError, TimeoutError } from 'rxjs';
import { catchError, timeout } from 'rxjs/operators';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000),
      catchError((err) => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException());
        }
        return throwError(() => err);
      }),
    );
  }
}
```
