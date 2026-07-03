---
title: Use Typed DTO Classes for Request Bodies
impact: HIGH
tags: dto,validation,body,class-validator,type-safety
---

## Use Typed DTO Classes for Request Bodies

Define `@Body` payloads as DTO **classes**, not interfaces or `any`. Interfaces are erased at compile time, so Nest can't reference them at runtime for validation. Classes survive transpilation, letting `ValidationPipe` enforce `class-validator` rules and transform the payload into a real instance.

**Incorrect:**

```typescript
@Post()
create(@Body() body: any) {
  return this.usersService.create(body);
}
```

**Correct:**

```typescript
// create-user.dto.ts
export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}

// users.controller.ts (with app.useGlobalPipes(new ValidationPipe()))
@Post()
create(@Body() dto: CreateUserDto): Promise<User> {
  return this.usersService.create(dto);
}
```
