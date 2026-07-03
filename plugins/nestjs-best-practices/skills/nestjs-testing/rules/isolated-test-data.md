---
title: Fresh fixtures per test; reset mocks and tear down between tests
impact: HIGH
tags: isolation,fixtures,teardown,lifecycle,jest
---

## Fresh fixtures per test; reset mocks and tear down between tests

Shared, mutated state is the top cause of order-dependent, flaky suites. Build fixtures
inside each test (or in `beforeEach`) so one test never sees another's leftovers. Reset
mocks between tests with `jest.clearAllMocks()` (or `clearMocks: true` in Jest config) so
call counts and implementations don't bleed across cases. For e2e, recreate or clean the
app/DB in `beforeEach`/`afterEach` and always `await app.close()` in `afterAll` to free
ports and DB connections.

**Incorrect:**

```typescript
// Module-level mutable fixture shared across tests, no reset.
const cats = [{ id: 1, name: 'Felix' }];

it('adds a cat', () => {
  service.create({ name: 'Milo' }); // mutates shared `cats`
  expect(cats).toHaveLength(2);
});

it('starts empty', () => {
  expect(cats).toHaveLength(1); // fails — polluted by the previous test
});
```

**Correct:**

```typescript
describe('CatsService', () => {
  let service: CatsService;
  const repo = { find: jest.fn(), save: jest.fn() };

  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
      providers: [
        CatsService,
        { provide: getRepositoryToken(Cat), useValue: repo },
      ],
    }).compile();
    service = moduleRef.get(CatsService);
  });

  afterEach(() => {
    jest.clearAllMocks(); // reset call history and implementations
  });

  it('saves a new cat', async () => {
    const input = { name: 'Milo' }; // fresh fixture, scoped to this test
    repo.save.mockResolvedValue({ id: 2, ...input });
    await expect(service.create(input)).resolves.toEqual({ id: 2, name: 'Milo' });
  });
});
```
