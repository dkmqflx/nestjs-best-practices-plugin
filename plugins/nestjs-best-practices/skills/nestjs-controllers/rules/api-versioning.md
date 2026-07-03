---
title: Version Evolving Public APIs
impact: LOW
tags: versioning,api,backwards-compatibility
---

## Version Evolving Public APIs

For public APIs that must evolve without breaking existing clients, enable Nest's built-in versioning instead of hand-rolling version prefixes. Call `enableVersioning` once, then attach `@Version` to controllers or individual routes. `VersioningType.URI` is the most common and cache-friendly form.

**Incorrect:**

```typescript
// Versions baked into the path prefix, no framework support
@Controller('v1/users')
export class UsersV1Controller {}

@Controller('v2/users')
export class UsersV2Controller {}
```

**Correct:**

```typescript
// main.ts
app.enableVersioning({ type: VersioningType.URI }); // -> /v1/users

// users-v1.controller.ts
@Controller({ path: 'users', version: '1' })
export class UsersV1Controller {
  @Get() findAll() {}
}

// users-v2.controller.ts
@Controller({ path: 'users', version: '2' })
export class UsersV2Controller {
  @Get() findAll() {}
}
```
