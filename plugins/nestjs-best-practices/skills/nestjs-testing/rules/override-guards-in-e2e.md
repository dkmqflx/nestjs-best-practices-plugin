---
title: Bypass auth in e2e with overrideGuard / overrideProvider
impact: HIGH
tags: e2e,guards,auth,override,supertest
---

## Bypass auth in e2e with overrideGuard / overrideProvider

When e2e tests target business endpoints, real authentication guards get in the way —
you don't want to mint valid JWTs for every request. The `TestingModule` builder exposes
`overrideGuard()` (and `overrideProvider()`, `overrideInterceptor()`, `overrideFilter()`)
to swap them for stubs before `.compile()`. Override the guard with one that always
allows access, and optionally inject a fake user onto the request so downstream handlers
behave. Keep a few tests that exercise the real guard so auth itself stays covered.

**Incorrect:**

```typescript
// Forced to generate and attach a real token on every request just to
// get past the guard — couples every test to the auth implementation.
it('GET /cats', () => {
  const token = jwtService.sign({ sub: 1 });
  return request(app.getHttpServer())
    .get('/cats')
    .set('Authorization', `Bearer ${token}`)
    .expect(200);
});
```

**Correct:**

```typescript
import { CanActivate, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '../src/auth/auth.guard';

const allowAll: CanActivate = {
  canActivate: (context: ExecutionContext) => {
    const req = context.switchToHttp().getRequest();
    req.user = { id: 1, roles: ['admin'] }; // stub the authenticated user
    return true;
  },
};

const moduleRef = await Test.createTestingModule({
  imports: [AppModule],
})
  .overrideGuard(AuthGuard)
  .useValue(allowAll)
  .compile();

app = moduleRef.createNestApplication();
await app.init();

it('GET /cats', () => {
  return request(app.getHttpServer()).get('/cats').expect(200);
});
```
