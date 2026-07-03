---
title: Index Queried Columns and Paginate Large Results
impact: HIGH
tags: typeorm, prisma, index, pagination, take, skip, performance
---

## Index Queried Columns and Paginate Large Results

Columns used in `WHERE`, `ORDER BY`, or join conditions need indexes, or every query becomes a full table scan that degrades as data grows. Likewise, never return an unbounded list — always paginate with `take`/`skip` (or keyset pagination for large offsets) so a single request can't load a million rows. Add `@Index()` to the entity (it ships in the migration) and cap result sizes at the query.

**Incorrect:**

```typescript
@Entity()
export class Order {
  @Column() customerId: string; // queried constantly, but no index
}

// Returns the entire table — unbounded result set
async list() {
  return this.orders.find({ where: { customerId } });
}
```

**Correct:**

```typescript
@Entity()
export class Order {
  @Index()
  @Column()
  customerId: string; // indexed -> migration adds the DB index
}

// Bounded, paginated query
async list(customerId: string, page = 1, limit = 20) {
  const [items, total] = await this.orders.findAndCount({
    where: { customerId },
    order: { createdAt: 'DESC' },
    take: limit,
    skip: (page - 1) * limit,
  });
  return { items, total, page, limit };
}
```

> Prisma: add `@@index([customerId])` in the schema and paginate with `take`/`skip` (or `cursor` for keyset pagination). Mongoose: declare `index: true` on the path or `schema.index({ customerId: 1 })`, and use `.limit()`/`.skip()`.

Reference: https://docs.nestjs.com/techniques/database
