---
title: Define DTOs as Classes, Not Interfaces
impact: CRITICAL
tags: validation,dto,classes,metadata,runtime
---

## Define DTOs as Classes, Not Interfaces

TypeScript interfaces (and `import type`) are erased at compile time, so `ValidationPipe` has no metadata to validate against and no constructor to instantiate. DTOs must be concrete classes, imported as values.

**Incorrect:**

```typescript
// Erased at runtime — ValidationPipe sees nothing to validate
export interface CreateUserDto {
  email: string;
  password: string;
}
```

**Correct:**

```typescript
import { IsEmail, IsNotEmpty } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsNotEmpty()
  password: string;
}
```
