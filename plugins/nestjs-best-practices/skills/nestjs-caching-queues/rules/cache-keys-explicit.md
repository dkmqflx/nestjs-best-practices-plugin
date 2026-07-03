---
title: Use Deterministic, Namespaced Cache Keys and Invalidate on Writes
impact: HIGH
tags: caching, cache-keys, invalidation, consistency, namespacing
---

## Use Deterministic, Namespaced Cache Keys and Invalidate on Writes

A cache is only correct if a write to the underlying data also updates or removes the cached copy. Two failures dominate in practice: (1) **ambiguous keys** — a bare `id` or a stringified object collides across entities or varies run-to-run, so reads miss and writes can't find the entry to delete; (2) **read-only caching** — the service caches on read but never calls `del` on update/delete, so callers see stale data until the TTL expires.

Build keys deterministically from a stable namespace plus the identifying inputs (`entity:id`, `entity:id:variant`). On every mutating operation, delete (or overwrite) exactly the keys that operation affects, in the same method that performs the write — don't rely on TTL alone to paper over a missing invalidation.

**Incorrect:**

```typescript
async findOne(id: number) {
  // Bare numeric key — collides with other entities, and nothing invalidates it.
  const cached = await this.cacheManager.get(`${id}`);
  if (cached) return cached;
  const user = await this.repo.findById(id);
  await this.cacheManager.set(`${id}`, user, 60_000);
  return user;
}

async update(id: number, dto: UpdateUserDto) {
  // Writes the DB but never touches the cache → reads stay stale until TTL.
  return this.repo.update(id, dto);
}
```

**Correct:**

```typescript
private key(id: number) {
  return `user:${id}`; // namespaced + deterministic
}

async findOne(id: number) {
  const cached = await this.cacheManager.get<User>(this.key(id));
  if (cached !== undefined) return cached; // v7+ returns undefined on miss
  const user = await this.repo.findById(id);
  await this.cacheManager.set(this.key(id), user, 60_000);
  return user;
}

async update(id: number, dto: UpdateUserDto) {
  const user = await this.repo.update(id, dto);
  await this.cacheManager.del(this.key(id)); // invalidate the exact key on write
  return user;
}
```
