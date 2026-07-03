---
title: Separate Authentication from Authorization
impact: HIGH
tags: guards,authentication,authorization,separation-of-concerns
---

## Separate Authentication from Authorization

Authentication (who you are) and authorization (what you may do) are different
concerns and fail differently: a missing/invalid credential is a `401`, while a
valid user lacking permission is a `403`. Cramming both into one guard couples
token verification to business roles, makes the guard untestable in isolation,
and tends to produce the wrong status code. Use one guard to verify identity and
attach `req.user`, and a separate guard to read route metadata and check
permissions. Guards run in declaration order, so the authn guard runs first and
the authz guard can rely on `req.user` already being populated.

**Incorrect:**

```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private jwt: JwtService, private reflector: Reflector) {}

  async canActivate(ctx: ExecutionContext): Promise<boolean> {
    const req = ctx.switchToHttp().getRequest();
    const user = await this.jwt.verifyAsync(extractToken(req)); // authn
    req.user = user;

    // authz tangled into the same guard — wrong layer, wrong 401 vs 403
    const roles = this.reflector.get<string[]>('roles', ctx.getHandler());
    if (roles && !roles.some((r) => user.roles?.includes(r))) {
      throw new UnauthorizedException(); // should be Forbidden
    }
    return true;
  }
}
```

**Correct:**

```typescript
// authn guard: only verifies identity and attaches the user
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}

// authz guard: only checks permissions against the already-attached user
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(ctx: ExecutionContext): boolean {
    const roles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      ctx.getHandler(),
      ctx.getClass(),
    ]);
    if (!roles) return true;
    const { user } = ctx.switchToHttp().getRequest();
    return roles.some((r) => user.roles?.includes(r)); // false -> 403
  }
}

// JwtAuthGuard runs first, RolesGuard second
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles(Role.Admin)
@Get('admin')
adminOnly() {}
```
