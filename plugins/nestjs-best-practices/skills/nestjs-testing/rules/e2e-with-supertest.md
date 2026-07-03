---
title: Exercise the whole app over HTTP with supertest
impact: HIGH
tags: e2e,supertest,createnestapplication,http
---

## Exercise the whole app over HTTP with supertest

E2e tests verify the full request pipeline — routing, pipes, guards, interceptors,
serialization — by sending real HTTP requests. Build a `TestingModule` that imports the
application module(s), create an app with `moduleRef.createNestApplication()`, call
`await app.init()`, then fire requests with `supertest` against `app.getHttpServer()`.
Apply the same global pipes/filters you register in `main.ts` so the test matches
production behavior. Always close the app in `afterAll` to release the port and handles.

**Incorrect:**

```typescript
// Calls the controller method directly — skips routing, pipes, guards,
// and serialization, so it proves nothing about the HTTP contract.
it('GET /cats', async () => {
  const controller = moduleRef.get(CatsController);
  expect(await controller.findAll()).toBeDefined();
});
```

**Correct:**

```typescript
import * as request from 'supertest';
import { INestApplication, ValidationPipe } from '@nestjs/common';
import { Test } from '@nestjs/testing';
import { AppModule } from '../src/app.module';

describe('Cats (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleRef.createNestApplication();
    app.useGlobalPipes(new ValidationPipe()); // mirror main.ts
    await app.init();
  });

  it('GET /cats', () => {
    return request(app.getHttpServer())
      .get('/cats')
      .expect(200)
      .expect((res) => {
        expect(Array.isArray(res.body)).toBe(true);
      });
  });

  afterAll(async () => {
    await app.close();
  });
});
```
