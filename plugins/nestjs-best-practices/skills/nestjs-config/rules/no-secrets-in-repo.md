---
title: Keep secrets out of the repo; commit only .env.example
impact: CRITICAL
tags: secrets, gitignore, env-example, security
---

## Keep secrets out of the repo; commit only .env.example

Never commit real secrets (DB passwords, API keys, JWT secrets) to version control —
git history is forever, and a leaked key means rotating credentials. Keep `.env` (and
`.env.*`) git-ignored, supply secrets via the environment or a secret manager in
production, and commit only a `.env.example` that documents required keys with placeholder
values.

**Incorrect:**

```bash
# .env committed to the repo
DATABASE_URL=postgres://admin:SuperSecret123@prod-db:5432/app
JWT_SECRET=8f3b9a2c-real-production-secret

# .gitignore does NOT list .env → secrets leak into git history
```

**Correct:**

```bash
# .gitignore
.env
.env.*
!.env.example

# .env.example (committed) — documents keys, no real values
DATABASE_URL=postgres://user:password@localhost:5432/app
JWT_SECRET=change-me

# Production reads from real environment / secret manager, never from a committed file:
ConfigModule.forRoot({ isGlobal: true, ignoreEnvFile: process.env.NODE_ENV === 'production' });
```
