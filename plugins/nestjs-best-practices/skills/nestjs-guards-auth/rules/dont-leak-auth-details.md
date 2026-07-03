---
title: Return Generic Auth Errors
impact: HIGH
tags: authentication,security,error-handling,enumeration
---

## Return Generic Auth Errors

Distinguishing "user not found" from "wrong password" lets an attacker enumerate
valid accounts and focus password-guessing on real users. Likewise, echoing
internal details (stack traces, JWT verification messages, which claim failed)
hands an attacker a roadmap. For authentication failures always return a single
generic `401 Unauthorized` regardless of the cause; for authorization failures
return a generic `403 Forbidden` without explaining the missing permission. Log
the specific reason server-side, but never send it to the client.

**Incorrect:**

```typescript
async signIn(username: string, pass: string) {
  const user = await this.users.findOne(username);
  if (!user) {
    throw new NotFoundException(`No user named ${username}`); // confirms account absence
  }
  if (!(await bcrypt.compare(pass, user.password))) {
    throw new UnauthorizedException('Incorrect password'); // confirms account exists
  }
  // ...
}
```

**Correct:**

```typescript
async signIn(username: string, pass: string) {
  const user = await this.users.findOne(username);
  const ok = user && (await bcrypt.compare(pass, user.password));
  if (!ok) {
    this.logger.warn(`Failed login for ${username}`); // detail stays server-side
    throw new UnauthorizedException(); // same generic 401 either way
  }
  return this.issueToken(user);
}

// authorization failure: generic 403, no mention of the required role
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(ctx: ExecutionContext): boolean {
    // returning false yields a plain ForbiddenException — don't add a reason
    return this.hasRequiredRole(ctx);
  }
}
```
