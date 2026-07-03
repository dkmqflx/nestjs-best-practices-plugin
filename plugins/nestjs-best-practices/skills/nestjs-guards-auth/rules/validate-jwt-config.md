---
title: Load a Strong JWT Config from the Environment
impact: CRITICAL
tags: authentication,jwt,configuration,secrets
---

## Load a Strong JWT Config from the Environment

A JWT is only as trustworthy as its signing secret and verification settings. A
hard-coded or weak secret in source control lets anyone forge tokens; a missing
expiry produces tokens valid forever. Source the secret from configuration/env
(via `ConfigModule`/`ConfigService`), fail fast if it is absent (`getOrThrow`),
keep it long and random, set a short `expiresIn` for access tokens, and ensure
the verifier checks both signature and expiration (`ignoreExpiration: false`).
Use `registerAsync` so the secret is read at runtime, not baked into the build.

**Incorrect:**

```typescript
// constants committed to the repo
export const jwtConstants = { secret: 'secret123' };

JwtModule.register({
  secret: jwtConstants.secret, // weak, hard-coded, in version control
  // no signOptions -> tokens never expire
});

// strategy that skips expiry verification
super({
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  ignoreExpiration: true, // accepts expired tokens
  secretOrKey: jwtConstants.secret,
});
```

**Correct:**

```typescript
// secret read from env at runtime; missing value crashes startup loudly
JwtModule.registerAsync({
  inject: [ConfigService],
  useFactory: (config: ConfigService) => ({
    secret: config.getOrThrow<string>('JWT_SECRET'), // long, random, from env
    signOptions: { expiresIn: '15m' }, // short-lived access token
  }),
});

// strategy verifies signature AND expiration
super({
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  ignoreExpiration: false,
  secretOrKey: config.getOrThrow<string>('JWT_SECRET'),
});
```
