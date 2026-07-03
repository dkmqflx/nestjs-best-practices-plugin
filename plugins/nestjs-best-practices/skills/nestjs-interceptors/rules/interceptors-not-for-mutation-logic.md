---
title: Keep business logic out of interceptors
impact: HIGH
tags: separation-of-concerns,service,architecture,cross-cutting
---

## Keep business logic out of interceptors

Interceptors exist for *cross-cutting* concerns — transforming, logging,
timing, caching — that apply uniformly across many handlers. Domain rules
(creating records, charging cards, mutating state) belong in services injected
into controllers. Burying business logic in `intercept()` hides it from the
route, makes it untestable in isolation, and couples unrelated handlers to
behavior they never opted into. Keep `intercept()` thin: pipe RxJS operators
onto `next.handle()` and return.

**Incorrect:**

```typescript
// Persisting a side effect for every wrapped route — invisible business logic.
@Injectable()
export class AuditInterceptor implements NestInterceptor {
  constructor(private readonly orders: OrdersService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      tap((order) => this.orders.chargeAndFulfill(order)), // domain logic!
    );
  }
}
```

**Correct:**

```typescript
// Interceptor only logs; the controller calls the service for domain work.
@Injectable()
export class AuditInterceptor implements NestInterceptor {
  private readonly logger = new Logger(AuditInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(tap((order) => this.logger.log(`order ${order.id}`)));
  }
}

@Controller('orders')
export class OrdersController {
  constructor(private readonly orders: OrdersService) {}

  @Post()
  create(@Body() dto: CreateOrderDto) {
    return this.orders.chargeAndFulfill(dto); // business logic lives here
  }
}
```
