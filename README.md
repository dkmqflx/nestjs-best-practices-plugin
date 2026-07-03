# nestjs-best-practices-plugin

A [Claude Code](https://claude.com/claude-code) plugin bundling 12 NestJS best-practice skills (90 rules total), grounded in the official [NestJS docs](https://docs.nestjs.com). Each skill follows a consistent format: an `impact`-tagged rule index plus incorrect/correct TypeScript examples.

## Install

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
