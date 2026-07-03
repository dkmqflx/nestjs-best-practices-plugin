---
title: Transform responses with map()
impact: HIGH
tags: response,transform,map,rxjs,envelope
---

## Transform responses with map()

Use the RxJS `map()` operator on `next.handle()` to wrap every handler's return
value in a consistent envelope (e.g. `{ data }`). Doing it in an interceptor
keeps controllers returning plain domain objects and guarantees a uniform
response shape across the whole app. Make the interceptor generic
(`NestInterceptor<T, Response<T>>`) so the response type stays accurate.

**Incorrect:**

```typescript
// Each controller hand-wraps its payload — easy to drift and forget.
@Get()
findAll() {
  const cats = this.catsService.findAll();
  return { data: cats }; // repeated in every handler
}
```

**Correct:**

```typescript
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
  data: T;
}

@Injectable()
export class TransformInterceptor<T>
  implements NestInterceptor<T, Response<T>>
{
  intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Observable<Response<T>> {
    return next.handle().pipe(map((data) => ({ data })));
  }
}
```
