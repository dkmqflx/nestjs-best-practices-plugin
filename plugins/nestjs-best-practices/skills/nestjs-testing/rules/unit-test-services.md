---
title: Test service logic against mocked repos, not a real DB
impact: HIGH
tags: unit,services,mocking,database,fast
---

## Test service logic against mocked repos, not a real DB

Service unit tests should verify business logic — branching, mapping, error handling —
not database behavior. Drive the mocked repository's return values to set up each case
(found, not found, error) and assert what the service does with them. This keeps tests
fast and deterministic, with no DB to spin up or reset. Save real persistence checks for
e2e or integration tests.

**Incorrect:**

```typescript
it('throws when cat is missing', async () => {
  // Relies on a real, seeded database — slow and flaky.
  await expect(service.findOne(999)).rejects.toThrow(NotFoundException);
});
```

**Correct:**

```typescript
describe('CatsService', () => {
  let service: CatsService;
  const repo = { findOne: jest.fn() };

  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
      providers: [
        CatsService,
        { provide: getRepositoryToken(Cat), useValue: repo },
      ],
    }).compile();
    service = moduleRef.get(CatsService);
  });

  it('returns the cat when found', async () => {
    repo.findOne.mockResolvedValue({ id: 1, name: 'Felix' });
    await expect(service.findOne(1)).resolves.toEqual({ id: 1, name: 'Felix' });
  });

  it('throws NotFoundException when missing', async () => {
    repo.findOne.mockResolvedValue(null);
    await expect(service.findOne(999)).rejects.toThrow(NotFoundException);
  });
});
```
