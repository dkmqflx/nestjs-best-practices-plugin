---
title: Map ORM Entities to DTOs; Don't Expose Persistence Models
impact: HIGH
tags: typeorm, dto, serialization, security, coupling, class-transformer
---

## Map ORM Entities to DTOs; Don't Expose Persistence Models

Returning ORM entities straight from a controller couples your public API to the database schema and risks leaking internal fields (password hashes, soft-delete flags, foreign keys). Define a response DTO and map to it, or annotate the entity with `class-transformer` and enable a serialization interceptor so excluded fields are stripped.

**Incorrect:**

```typescript
// Returns the raw entity — passwordHash and internal columns leak to the client
@Get(':id')
async findOne(@Param('id') id: string): Promise<User> {
  return this.usersService.findOne(id); // includes passwordHash, deletedAt, ...
}
```

**Correct:**

```typescript
// Explicit response DTO — only intended fields cross the boundary
export class UserResponseDto {
  id: string;
  name: string;
  email: string;
}

@Get(':id')
async findOne(@Param('id') id: string): Promise<UserResponseDto> {
  const user = await this.usersService.findOne(id);
  return { id: user.id, name: user.name, email: user.email };
}

// Alternative: class-transformer + ClassSerializerInterceptor
export class User {
  @Exclude() passwordHash: string; // stripped on serialization
}
// app: app.useGlobalInterceptors(new ClassSerializerInterceptor(app.get(Reflector)));
```

Reference: https://docs.nestjs.com/techniques/serialization
