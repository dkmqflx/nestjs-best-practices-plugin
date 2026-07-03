---
title: Return Promises or Observables From Handlers
impact: MEDIUM
tags: async,promises,observables,response-handling
---

## Return Promises or Observables From Handlers

Nest resolves any `Promise` or RxJS `Observable` a handler returns and serializes the final value automatically. Just `return` the async result (or use `async/await`) — don't manually await and write to a response object, which bypasses Nest's response pipeline and error handling.

**Incorrect:**

```typescript
@Get(':id')
async findOne(@Param('id') id: string, @Res() res: Response) {
  const user = await this.usersService.findOne(id);
  res.json(user); // opts out of interceptors, serialization, exception layer
}
```

**Correct:**

```typescript
@Get(':id')
findOne(@Param('id') id: string): Promise<User> {
  return this.usersService.findOne(id); // Nest awaits and serializes it
}
```
