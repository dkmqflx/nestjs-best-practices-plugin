---
title: Prefer Param Decorators Over Raw @Req/@Res
impact: HIGH
tags: params,query,body,request,response,decorators
---

## Prefer Param Decorators Over Raw @Req/@Res

Use `@Param`, `@Query`, and `@Body` to extract exactly what a handler needs — this keeps signatures explicit and testable. Injecting `@Res()` puts Nest in library-specific mode: you lose compatibility with features that rely on standard response handling, such as interceptors and the `@HttpCode()` / `@Header()` decorators. Reach for the raw request only when you genuinely need framework internals.

**Incorrect:**

```typescript
@Get(':id')
findOne(@Req() req: Request, @Res() res: Response) {
  const { id } = req.params;
  const { verbose } = req.query;
  res.send(this.usersService.findOne(id, verbose === 'true'));
}
```

**Correct:**

```typescript
@Get(':id')
findOne(
  @Param('id') id: string,
  @Query('verbose') verbose?: string,
): Promise<User> {
  return this.usersService.findOne(id, verbose === 'true');
}
```
