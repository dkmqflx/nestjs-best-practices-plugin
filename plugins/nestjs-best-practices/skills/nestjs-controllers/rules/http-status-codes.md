---
title: Set Correct HTTP Status Codes
impact: MEDIUM
tags: status-codes,httpcode,rest,responses
---

## Set Correct HTTP Status Codes

Nest responses default to 200, except `POST` handlers which default to 201. When the real semantics differ — a `POST` that doesn't create a resource, or a `DELETE`/`PUT` returning no body — override it with `@HttpCode(...)` so clients get an accurate status. Prefer the `HttpStatus` enum over magic numbers.

**Incorrect:**

```typescript
@Delete(':id')
remove(@Param('id') id: string): Promise<void> {
  return this.usersService.remove(id); // returns 200 with empty body
}
```

**Correct:**

```typescript
@Delete(':id')
@HttpCode(HttpStatus.NO_CONTENT) // 204
remove(@Param('id') id: string): Promise<void> {
  return this.usersService.remove(id);
}
```
