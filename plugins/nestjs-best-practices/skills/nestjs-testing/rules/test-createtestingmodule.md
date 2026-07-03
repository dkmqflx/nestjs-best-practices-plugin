---
title: Build the unit under test with Test.createTestingModule
impact: CRITICAL
tags: unit,testing-module,di,setup
---

## Build the unit under test with Test.createTestingModule

NestJS provides `Test.createTestingModule()` to assemble an isolated DI container for a
test. It wires up only the providers you declare, resolves dependencies through Nest's
injector, and hands back a `TestingModule` after `.compile()`. Manually `new`-ing a
controller or service bypasses the framework, forces you to pass constructor args by
hand, and breaks the moment the constructor signature changes. Always resolve instances
from the compiled module with `module.get()` (or `module.resolve()` for request/transient
scoped providers).

**Incorrect:**

```typescript
describe('CatsController', () => {
  // Hand-wired — brittle, ignores Nest DI, breaks on constructor changes.
  const catsService = new CatsService(new CatsRepository());
  const catsController = new CatsController(catsService);

  it('returns cats', async () => {
    expect(await catsController.findAll()).toBeDefined();
  });
});
```

**Correct:**

```typescript
import { Test, TestingModule } from '@nestjs/testing';

describe('CatsController', () => {
  let controller: CatsController;
  let service: CatsService;

  beforeEach(async () => {
    const moduleRef: TestingModule = await Test.createTestingModule({
      controllers: [CatsController],
      providers: [CatsService],
    }).compile();

    controller = moduleRef.get(CatsController);
    service = moduleRef.get(CatsService);
  });

  it('returns cats', async () => {
    jest.spyOn(service, 'findAll').mockResolvedValue(['test']);
    expect(await controller.findAll()).toEqual(['test']);
  });
});
```
