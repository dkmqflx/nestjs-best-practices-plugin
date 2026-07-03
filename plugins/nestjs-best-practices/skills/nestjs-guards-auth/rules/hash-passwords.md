---
title: Hash Passwords with bcrypt or argon2
impact: CRITICAL
tags: authentication,passwords,hashing,security
---

## Hash Passwords with bcrypt or argon2

Never store passwords in plaintext or compare them as plain strings — a database
leak then exposes every user's credential directly, and string comparison is
vulnerable to timing attacks. Hash passwords with a slow, salted, adaptive
algorithm (`bcrypt` or `argon2`) at registration, store only the hash, and verify
at login with the library's constant-time `compare`/`verify`. Salting is handled
by these libraries automatically. Use a work factor of at least 10–12 for bcrypt.

**Incorrect:**

```typescript
@Injectable()
export class AuthService {
  async signUp(dto: SignUpDto) {
    // plaintext stored verbatim
    return this.users.create({ ...dto, password: dto.password });
  }

  async validateUser(username: string, pass: string) {
    const user = await this.users.findOne(username);
    if (user?.password !== pass) throw new UnauthorizedException(); // plaintext compare
    return user;
  }
}
```

**Correct:**

```typescript
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  async signUp(dto: SignUpDto) {
    const passwordHash = await bcrypt.hash(dto.password, 12); // salted + slow
    return this.users.create({ ...dto, password: passwordHash });
  }

  async validateUser(username: string, pass: string) {
    const user = await this.users.findOne(username);
    // constant-time comparison; never throw before this to avoid leaking timing
    if (!user || !(await bcrypt.compare(pass, user.password))) {
      throw new UnauthorizedException();
    }
    const { password, ...result } = user;
    return result; // never return the hash
  }
}
```
