---
title: Enable Automatic Payload Transformation
impact: HIGH
tags: validation,transform,class-transformer,types,coercion
---

## Enable Automatic Payload Transformation

By default the request body is a plain object, not an instance of your DTO class, and path/query params arrive as strings. `transform: true` instantiates the DTO class and coerces primitive types declared by the handler signature.

**Incorrect:**

```typescript
app.useGlobalPipes(new ValidationPipe());

@Get(':id')
findOne(@Param('id') id: number) {
  // id is actually the string "42" — typeof id === 'string'
}
```

**Correct:**

```typescript
app.useGlobalPipes(new ValidationPipe({ transform: true }));

@Get(':id')
findOne(@Param('id') id: number) {
  // id is the number 42, and body becomes a real DTO instance
}
```
