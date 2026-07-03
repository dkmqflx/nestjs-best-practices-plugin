---
title: Annotate Every DTO Field
impact: HIGH
tags: validation,class-validator,decorators,dto
---

## Annotate Every DTO Field

`ValidationPipe` only enforces constraints expressed as `class-validator` decorators. An un-decorated field is unchecked — and with `whitelist: true` it is silently stripped. Decorate every field, marking optional ones with `@IsOptional`.

**Incorrect:**

```typescript
export class CreateUserDto {
  email: string; // no decorator: not validated (or stripped by whitelist)
  age: number;
}
```

**Correct:**

```typescript
import { IsEmail, IsInt, Min, IsOptional } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsOptional()
  @IsInt()
  @Min(0)
  age?: number;
}
```
