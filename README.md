# nestjs-best-practices-plugin

[![Release](https://img.shields.io/badge/release-v0.1.0-blue)](https://github.com/dkmqflx/nestjs-best-practices-plugin/releases/tag/nestjs-best-practices--v0.1.0)

12 NestJS best-practice skills (90 rules total), grounded in the official [NestJS docs](https://docs.nestjs.com). Each skill follows a consistent format: an `impact`-tagged rule index plus incorrect/correct TypeScript examples.

## Supported Coding Agents

These skills can be installed via [`npx skills`](https://github.com/vercel-labs/skills) for any agent that supports the [Agent Skills specification](https://skills.sh), including Claude Code, Codex, Gemini CLI, Cursor, Windsurf, and more.

## Install

### Quick Install (any supported agent)

Using [`npx skills`](https://github.com/vercel-labs/skills):

```bash
# Current project
npx skills add dkmqflx/nestjs-best-practices-plugin --skill '*' --yes

# All projects (global)
npx skills add dkmqflx/nestjs-best-practices-plugin --skill '*' --yes --global
```

To target a specific agent:

```bash
npx skills add dkmqflx/nestjs-best-practices-plugin --agent codex --skill '*' --yes
npx skills add dkmqflx/nestjs-best-practices-plugin --agent gemini-cli --skill '*' --yes
```

### Claude Code Plugin

```
/plugin marketplace add dkmqflx/nestjs-best-practices-plugin
/plugin install nestjs-best-practices@nestjs-best-practices-plugin
```

Then run `/reload-plugins`. Skills are model-invoked automatically based on context (e.g. writing a `@Controller`, a `@Module`, a guard, etc.) — you don't need to call them by name.

## Skills

| Skill | Description |
|-------|-------------|
| `nestjs-modules-di` | Modules & dependency injection — feature boundaries, provider scopes, dynamic modules, circular deps |
| `nestjs-controllers` | Controllers & routing — thin controllers, DTOs, param decorators, serialization, versioning |
| `nestjs-validation-pipes` | Validation & pipes — global `ValidationPipe`, whitelist, transform, class-validator, custom pipes |
| `nestjs-guards-auth` | Guards, auth & authorization — Passport/JWT, global guard + `@Public()`, RBAC, password hashing |
| `nestjs-interceptors` | Interceptors — response transform, logging, timeout, caching, global vs scoped binding |
| `nestjs-exception-filters` | Exception handling & filters — built-in HTTP exceptions, global filters, consistent error shape |
| `nestjs-config` | Configuration — global `ConfigModule`, env schema validation, typed/namespaced config |
| `nestjs-database` | Database & ORM (TypeORM/Prisma/Mongoose) — repositories, migrations, transactions, N+1, pagination |
| `nestjs-caching-queues` | Caching & queues — `CacheModule`, TTL/keys, BullMQ jobs, idempotent processors, retries |
| `nestjs-testing` | Testing — `Test.createTestingModule`, mocking providers, e2e with supertest, guard overrides |
| `nestjs-security` | Security hardening — helmet, throttler rate limiting, explicit CORS, CSRF, secrets, payload limits |
| `nestjs-performance-logging` | Performance & logging — Fastify adapter, compression, structured pino logging, health checks, graceful shutdown |

## Local development

```
claude --plugin-dir ./plugins/nestjs-best-practices
```

## License

MIT
