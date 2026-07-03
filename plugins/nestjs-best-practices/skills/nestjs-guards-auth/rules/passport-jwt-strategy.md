---
title: Use a Passport JWT Strategy for Authentication
impact: HIGH
tags: authentication,passport,jwt,strategy
---

## Use a Passport JWT Strategy for Authentication

For JWT auth, prefer `@nestjs/passport` with a `passport-jwt` strategy class
over hand-rolling token extraction and verification inside a guard. The strategy
centralizes how the token is extracted, configures signature and expiration
verification declaratively (`ignoreExpiration: false`), and its `validate()`
return value is what Passport attaches to `req.user`. Guards then become trivial
(`AuthGuard('jwt')`), and the verification logic is reusable and testable.
Always set `ignoreExpiration: false` so expired tokens are rejected, and source
`secretOrKey` from configuration rather than a literal.

**Incorrect:**

```typescript
// Re-implementing extraction + verification by hand in every guard
@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(private jwt: JwtService) {}

  async canActivate(ctx: ExecutionContext): Promise<boolean> {
    const req = ctx.switchToHttp().getRequest();
    const token = req.headers.authorization?.split(' ')[1];
    const payload = await this.jwt.verifyAsync(token); // no expiry strategy, ad hoc
    req.user = payload; // raw payload, no validate() step
    return true;
  }
}
```

**Correct:**

```typescript
// auth/jwt.strategy.ts
import { ExtractJwt, Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(config: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false, // reject expired tokens
      secretOrKey: config.getOrThrow<string>('JWT_SECRET'),
    });
  }

  // return value becomes req.user
  async validate(payload: { sub: string; username: string; roles: string[] }) {
    return { userId: payload.sub, username: payload.username, roles: payload.roles };
  }
}

// auth/jwt-auth.guard.ts
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}
```
