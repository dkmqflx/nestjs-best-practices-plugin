---
title: Load Secrets from Env, Never Hardcode
impact: CRITICAL
tags: secrets,env,config,git,credentials
---

## Load Secrets from Env, Never Hardcode

Hardcoded secrets — DB passwords, JWT signing keys, API tokens — leak the moment the repo is cloned, forked, or its history is inspected, and rotating them means a code change and redeploy. Load every secret from environment variables (or a secret manager) via `@nestjs/config`, validate they are present at boot, and keep `.env` out of git. A missing secret should fail startup loudly, not fall back to a default.

**Incorrect:**

```typescript
// jwt.config.ts — committed to the repo
export const jwtConfig = {
  secret: 'super-secret-key-123', // leaked to everyone with repo access
  expiresIn: '1h',
};
```

**Correct:**

```typescript
// app.module.ts — validate presence at boot, read from env
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: Joi.object({
        JWT_SECRET: Joi.string().min(32).required(),
        DATABASE_URL: Joi.string().uri().required(),
      }),
    }),
  ],
})
export class AppModule {}

// .gitignore must include:  .env
// Consume via configService.getOrThrow('JWT_SECRET')
```
