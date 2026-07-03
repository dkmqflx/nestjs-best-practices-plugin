---
title: Register a Global Auth Guard with @Public() Opt-Out
impact: HIGH
tags: guards,authentication,global-guard,reflector,metadata
---

## Register a Global Auth Guard with @Public() Opt-Out

Protecting routes by remembering to add `@UseGuards(JwtAuthGuard)` on every
controller is error-prone — one forgotten decorator leaves an endpoint open. The
safer default is "secure by default": register the auth guard globally via the
`APP_GUARD` provider so every route is protected, then explicitly opt specific
routes (login, health, signup) out with a `@Public()` metadata decorator that
the guard reads through `Reflector`. Use `getAllAndOverride` over both
`getHandler()` and `getClass()` so the decorator can be applied at the method or
controller level.

**Incorrect:**

```typescript
// Each controller must remember to add the guard — easy to miss one
@UseGuards(JwtAuthGuard)
@Controller('orders')
export class OrdersController {}

@Controller('payments') // forgot the guard -> wide open
export class PaymentsController {}
```

**Correct:**

```typescript
// auth/public.decorator.ts
import { SetMetadata } from '@nestjs/common';
export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// auth/jwt-auth.guard.ts
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(ctx: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      ctx.getHandler(),
      ctx.getClass(),
    ]);
    if (isPublic) return true;
    return super.canActivate(ctx);
  }
}

// app.module.ts — protect everything by default
@Module({
  providers: [{ provide: APP_GUARD, useClass: JwtAuthGuard }],
})
export class AppModule {}

// opt a single route out
@Public()
@Post('login')
login(@Body() dto: LoginDto) {}
```
