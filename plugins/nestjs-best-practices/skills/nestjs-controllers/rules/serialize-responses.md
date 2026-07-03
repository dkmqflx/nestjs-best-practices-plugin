---
title: Serialize Responses to Hide Sensitive Fields
impact: CRITICAL
tags: serialization,security,class-transformer,interceptors,exclude
---

## Serialize Responses to Hide Sensitive Fields

Returning an entity directly can leak sensitive fields like password hashes or tokens. Apply `ClassSerializerInterceptor` and mark sensitive properties with `@Exclude()` from `class-transformer`. The interceptor runs `instanceToPlain` on returned class instances, stripping excluded fields — so the handler must return an instance of the decorated class.

**Incorrect:**

```typescript
@Get(':id')
findOne(@Param('id') id: string): Promise<User> {
  // User has a `password` column -> serialized straight to the client
  return this.usersService.findOne(id);
}
```

**Correct:**

```typescript
// user.entity.ts
export class User {
  id: string;
  email: string;

  @Exclude()
  password: string;
}

// users.controller.ts
@UseInterceptors(ClassSerializerInterceptor)
@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id') id: string): Promise<User> {
    return this.usersService.findOne(id); // password stripped from output
  }
}
```
