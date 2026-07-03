---
title: Wrap Multi-Step Writes in a Transaction
impact: CRITICAL
tags: typeorm, prisma, transactions, queryrunner, atomicity, consistency
---

## Wrap Multi-Step Writes in a Transaction

When a single operation performs several writes that must all succeed or all fail (e.g. debit one account, credit another), run them in a transaction so a mid-way failure cannot leave the database in an inconsistent state. In TypeORM use `dataSource.transaction(...)` for the common case, or a `QueryRunner` when you need explicit control; commit on success and roll back on error.

**Incorrect:**

```typescript
// No transaction: if the second write throws, the first is already committed
async transfer(fromId: string, toId: string, amount: number) {
  await this.accounts.decrement({ id: fromId }, 'balance', amount);
  await this.accounts.increment({ id: toId }, 'balance', amount); // may fail -> money lost
}
```

**Correct:**

```typescript
// Managed transaction — auto commit/rollback
async transfer(fromId: string, toId: string, amount: number) {
  await this.dataSource.transaction(async (manager) => {
    await manager.decrement(Account, { id: fromId }, 'balance', amount);
    await manager.increment(Account, { id: toId }, 'balance', amount);
  });
}

// Explicit QueryRunner when you need fine control:
async transferExplicit(fromId: string, toId: string, amount: number) {
  const qr = this.dataSource.createQueryRunner();
  await qr.connect();
  await qr.startTransaction();
  try {
    await qr.manager.decrement(Account, { id: fromId }, 'balance', amount);
    await qr.manager.increment(Account, { id: toId }, 'balance', amount);
    await qr.commitTransaction();
  } catch (err) {
    await qr.rollbackTransaction();
    throw err;
  } finally {
    await qr.release(); // always release the connection
  }
}
```

> Prisma: use `prisma.$transaction([...])` for a batch, or `prisma.$transaction(async (tx) => { ... })` for interactive logic — and use `tx`, not the root client, inside the callback.

Reference: https://docs.nestjs.com/techniques/database
