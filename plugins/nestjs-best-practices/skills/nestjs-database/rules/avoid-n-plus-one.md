---
title: Load Relations Explicitly; Never Query Inside a Loop
impact: HIGH
tags: typeorm, prisma, relations, n+1, performance, eager-loading
---

## Load Relations Explicitly; Never Query Inside a Loop

The N+1 problem: one query fetches N rows, then the code issues another query per row to load a relation — N+1 round trips that crush performance under load. Fetch related data in a single query with `relations` (or a `leftJoinAndSelect` query builder) instead of looping. Avoid `eager: true` on entities globally; load relations per-query where you actually need them.

**Incorrect:**

```typescript
// 1 query for users + N queries for each user's orders
const users = await this.users.find();
for (const user of users) {
  user.orders = await this.orders.findBy({ userId: user.id }); // N extra queries
}
return users;
```

**Correct:**

```typescript
// Single query that joins the relation
const users = await this.users.find({
  relations: { orders: true },
});

// Or with the query builder for control over the join:
const users = await this.users
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.orders', 'order')
  .getMany();
```

> Prisma: pass `include: { orders: true }` (or `select`) so the relation is resolved in one round trip. Mongoose: use `.populate('orders')` rather than fetching referenced docs in a loop.

Reference: https://docs.nestjs.com/techniques/database
