---
title: Provide mock providers for a unit's dependencies
impact: CRITICAL
tags: unit,mocking,usevalue,providers,isolation
---

## Provide mock providers for a unit's dependencies

A unit test should exercise one class in isolation. Register the class under test as a
real provider, but supply test doubles for everything it depends on so you never touch a
database, network, or another module's logic. Use `useValue` with `jest.fn()` for each
dependency (referencing it by its injection token), or `useMocker()` to auto-mock all
unspecified dependencies at once. Provide mocks for injection tokens too — e.g. a TypeORM
repository registered via `getRepositoryToken(Cat)`.

**Incorrect:**

```typescript
// Pulls in the real service, which opens a real DB connection.
const moduleRef = await Test.createTestingModule({
  providers: [CatsService, CatsRepository, DatabaseConnection],
}).compile();
```

**Correct:**

```typescript
import { getRepositoryToken } from '@nestjs/typeorm';

const repoMock = {
  find: jest.fn(),
  findOne: jest.fn(),
  save: jest.fn(),
};

const moduleRef = await Test.createTestingModule({
  providers: [
    CatsService,
    { provide: getRepositoryToken(Cat), useValue: repoMock },
  ],
}).compile();

// Or auto-mock every unspecified dependency:
const autoMocked = await Test.createTestingModule({
  providers: [CatsService],
})
  .useMocker((token) => {
    if (token === getRepositoryToken(Cat)) {
      return { find: jest.fn().mockResolvedValue([]) };
    }
  })
  .compile();
```
