---
title: Hash Passwords and Disable x-powered-by
impact: CRITICAL
tags: passwords,bcrypt,argon2,hashing,fingerprinting,headers
---

## Hash Passwords and Disable x-powered-by

Two distinct hardening steps. First: never store passwords in plaintext or with a fast/general-purpose hash (MD5, SHA-256) — a DB leak then exposes every credential. Use a slow, salted password hash (`bcrypt` or `argon2`) so cracking is computationally expensive. Second: the default `X-Powered-By: Express` header tells attackers exactly what you run, helping them target known CVEs. Disable it (helmet removes it automatically; otherwise call `app.disable('x-powered-by')`).

**Incorrect:**

```typescript
// Plaintext (or fast-hash) password + leaky framework header
async register(dto: RegisterDto) {
  return this.users.create({ ...dto, password: dto.password }); // stored as-is
}
// No header hardening — responses advertise "X-Powered-By: Express"
```

**Correct:**

```typescript
// auth.service.ts — slow, salted hashing
import * as bcrypt from 'bcrypt';

async register(dto: RegisterDto) {
  const passwordHash = await bcrypt.hash(dto.password, 12);
  return this.users.create({ ...dto, password: passwordHash });
}

async verify(plain: string, hash: string) {
  return bcrypt.compare(plain, hash);
}

// main.ts — drop the fingerprinting header (Express)
// helmet() also removes it; otherwise:
app.getHttpAdapter().getInstance().disable('x-powered-by');
```
