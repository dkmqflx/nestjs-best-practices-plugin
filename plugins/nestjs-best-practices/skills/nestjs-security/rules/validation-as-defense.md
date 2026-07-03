---
title: Use a Global ValidationPipe with Whitelist
impact: HIGH
tags: validation,whitelist,mass-assignment,injection,dto
---

## Use a Global ValidationPipe with Whitelist

Without validation, clients control the shape of every request body — enabling injection payloads and mass-assignment (e.g. sneaking `isAdmin: true` into a user-update DTO). A global `ValidationPipe` with `whitelist: true` strips properties not declared on the DTO; `forbidNonWhitelisted: true` rejects them outright with a 400. Pair it with `class-validator` decorators on DTOs as the typed contract the API accepts.

**Incorrect:**

```typescript
// No global validation; extra fields flow straight into the entity
@Post()
create(@Body() dto: CreateUserDto) {
  return this.users.create(dto); // attacker can pass { isAdmin: true }
}
```

**Correct:**

```typescript
// main.ts
import { ValidationPipe } from '@nestjs/common';

app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,            // drop unknown properties
    forbidNonWhitelisted: true, // 400 if unknown properties present
    transform: true,            // coerce payloads to DTO types
  }),
);

// create-user.dto.ts
import { IsEmail, IsString, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsEmail() email: string;
  @IsString() @MinLength(8) password: string;
}
```
