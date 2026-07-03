---
title: Assert outputs and HTTP contracts, not internal call shapes
impact: MEDIUM
tags: behavior,assertions,contracts,refactor-safe
---

## Assert outputs and HTTP contracts, not internal call shapes

Tests that assert *how* a unit works internally — exact mock call counts, the precise
arguments passed to a repository — break on harmless refactors even when behavior is
unchanged. Prefer asserting *what* the unit produces: the returned value, the thrown
error, or the HTTP status and response body. Verifying a specific interaction is fair
only when that interaction is itself the contract (e.g. "an email was sent", "the payment
gateway was charged exactly once"); reserve `toHaveBeenCalledWith` for those cases.

**Incorrect:**

```typescript
it('creates a cat', async () => {
  await service.create({ name: 'Milo' });
  // Locks the test to internal implementation details.
  expect(repo.save).toHaveBeenCalledTimes(1);
  expect(repo.save).toHaveBeenCalledWith({ name: 'Milo', createdAt: expect.any(Date) });
});
```

**Correct:**

```typescript
it('creates and returns the cat', async () => {
  repo.save.mockResolvedValue({ id: 2, name: 'Milo' });
  // Assert the observable result, not how it was produced.
  await expect(service.create({ name: 'Milo' })).resolves.toEqual({
    id: 2,
    name: 'Milo',
  });
});

it('POST /cats returns 201 with the created resource', () => {
  return request(app.getHttpServer())
    .post('/cats')
    .send({ name: 'Milo' })
    .expect(201)
    .expect((res) => expect(res.body).toMatchObject({ name: 'Milo' }));
});

// Interaction assertion is justified when sending the email IS the contract:
it('sends a welcome email on signup', async () => {
  await service.signup({ email: 'a@b.com' });
  expect(mailer.send).toHaveBeenCalledWith(
    expect.objectContaining({ to: 'a@b.com' }),
  );
});
```
