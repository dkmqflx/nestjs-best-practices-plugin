---
title: Project Only the Columns You Need
impact: MEDIUM
tags: typeorm, prisma, select, projection, over-fetching, performance
---

## Project Only the Columns You Need

Fetching every column wastes I/O and memory and risks pulling sensitive fields (password hashes, tokens) into application memory where they may leak into responses or logs. Select only the fields a use case requires. This pairs with `separate-entities-from-dtos`: project at the query level and you have less to strip later.

**Incorrect:**

```typescript
// Loads every column, including password hash, just to show a name + email
async listUsers() {
  return this.users.find(); // SELECT * — over-fetching
}
```

**Correct:**

```typescript
// Project exactly the columns the screen needs
async listUsers() {
  return this.users.find({
    select: { id: true, name: true, email: true },
  });
}

// Query builder equivalent:
this.users
  .createQueryBuilder('user')
  .select(['user.id', 'user.name', 'user.email'])
  .getMany();
```

> Prefer marking secrets with `{ select: false }` on the entity column so they are never returned unless explicitly requested. Prisma: use `select: { id: true, name: true, email: true }`. Mongoose: use a projection like `.find({}, 'id name email')` or `.select('id name email')`.

Reference: https://docs.nestjs.com/techniques/database
