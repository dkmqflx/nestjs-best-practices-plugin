---
title: Drive RBAC with @Roles Metadata and a RolesGuard
impact: HIGH
tags: authorization,rbac,roles,reflector,metadata
---

## Drive RBAC with @Roles Metadata and a RolesGuard

Hard-coding role checks inside controller bodies scatters authorization logic,
duplicates it, and makes it invisible at the route signature. Instead, declare
required roles as metadata with a `@Roles()` decorator and centralize the check
in a single `RolesGuard` that reads that metadata via `Reflector`. Use a typed
`Role` enum (not magic strings), read with `getAllAndOverride` across
`getHandler()` and `getClass()`, and return `true` when no roles are required so
the guard is opt-in per route. The RolesGuard must run after the authn guard so
`req.user` is populated.

**Incorrect:**

```typescript
@Post('reports')
generateReport(@Req() req) {
  // authorization buried in the handler, repeated everywhere, string-typed
  if (req.user.role !== 'admin' && req.user.role !== 'manager') {
    throw new ForbiddenException();
  }
  return this.reports.generate();
}
```

**Correct:**

```typescript
// enums/role.enum.ts
export enum Role {
  User = 'user',
  Admin = 'admin',
}

// auth/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';
export const ROLES_KEY = 'roles';
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);

// auth/roles.guard.ts
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(ctx: ExecutionContext): boolean {
    const required = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      ctx.getHandler(),
      ctx.getClass(),
    ]);
    if (!required) return true; // no @Roles -> not restricted
    const { user } = ctx.switchToHttp().getRequest();
    return required.some((role) => user.roles?.includes(role));
  }
}

// usage — declarative and visible at the route
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles(Role.Admin)
@Post('reports')
generateReport() {
  return this.reports.generate();
}
```
