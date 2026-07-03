---
title: Coerce Params with Built-in Parse Pipes
impact: MEDIUM
tags: pipes,ParseIntPipe,ParseUUIDPipe,ParseBoolPipe,params
---

## Coerce Params with Built-in Parse Pipes

Route and query params arrive as strings. Built-in parse pipes both convert them to the correct type and reject malformed values with a 400 before the handler runs, so you never hand-parse or trust raw input.

**Incorrect:**

```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  const parsed = Number(id); // NaN on bad input, no automatic 400
  return this.service.findOne(parsed);
}
```

**Correct:**

```typescript
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  return this.service.findOne(id);
}

@Get()
list(@Query('active', ParseBoolPipe) active: boolean) {}

@Delete(':uuid')
remove(@Param('uuid', ParseUUIDPipe) uuid: string) {}
```
