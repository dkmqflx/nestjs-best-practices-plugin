---
title: Follow RESTful Resource Routing Conventions
impact: MEDIUM
tags: routing,rest,http-verbs,resource-naming
---

## Follow RESTful Resource Routing Conventions

Name resources with plural nouns and let the HTTP verb express the action — don't put verbs in the path. A controller prefix plus method decorators (`@Get`, `@Post`, `@Patch`, `@Delete`) produces predictable, consistent URLs that map cleanly onto CRUD.

**Incorrect:**

```typescript
@Controller('user')
export class UsersController {
  @Post('createUser') create() {}
  @Get('getUserById/:id') getOne() {}
  @Post('user/:id/delete') remove() {}
}
```

**Correct:**

```typescript
@Controller('users')
export class UsersController {
  @Post() create() {}              // POST   /users
  @Get() findAll() {}              // GET    /users
  @Get(':id') findOne() {}         // GET    /users/:id
  @Patch(':id') update() {}        // PATCH  /users/:id
  @Delete(':id') remove() {}       // DELETE /users/:id
}
```
