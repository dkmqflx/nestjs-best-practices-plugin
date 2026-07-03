---
title: Strongly Type req.user
impact: MEDIUM
tags: authentication,typescript,types,request-user
---

## Strongly Type req.user

By default `req.user` is untyped (`any`), so every downstream handler accesses
fields with no compile-time checking and silently trusts whatever the strategy
attached. Define an explicit `AuthUser` type that mirrors exactly what your
strategy's `validate()` returns, and type handlers and a custom decorator against
it. This catches typos, documents the auth contract, and makes it obvious that
`req.user` carries only validated claims (never the raw, unverified JWT payload).
The strategy's `validate()` is the single place that maps a payload to this type.

**Incorrect:**

```typescript
@Get('profile')
getProfile(@Req() req) {
  // req.user is `any`; req.user.isAdmin is a typo that compiles fine
  return { id: req.user.userId, admin: req.user.isAdmin };
}
```

**Correct:**

```typescript
// auth/auth-user.type.ts — mirrors JwtStrategy.validate() return
export interface AuthUser {
  userId: string;
  username: string;
  roles: Role[];
}

// auth/current-user.decorator.ts
export const CurrentUser = createParamDecorator(
  (_: unknown, ctx: ExecutionContext): AuthUser =>
    ctx.switchToHttp().getRequest().user,
);

// handler — fully typed, typos rejected at compile time
@Get('profile')
getProfile(@CurrentUser() user: AuthUser) {
  return { id: user.userId, roles: user.roles };
}
```
