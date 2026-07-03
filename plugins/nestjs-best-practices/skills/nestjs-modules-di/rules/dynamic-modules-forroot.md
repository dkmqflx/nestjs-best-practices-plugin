---
title: Configure Reusable Modules with forRoot / forRootAsync
impact: HIGH
tags: dynamic-modules, configuration, forRoot, providers
---

## Configure Reusable Modules with forRoot / forRootAsync

When a module needs runtime configuration (credentials, options), expose a static method returning a `DynamicModule` instead of hard-coding values. Convention: `forRoot`/`forRootAsync` for once-per-app setup, `register`/`registerAsync` for per-import config. Async variants accept `useFactory` so options can come from injected dependencies like `ConfigService`.

**Incorrect:**

```typescript
@Module({
  providers: [{ provide: 'DB_OPTIONS', useValue: { host: 'localhost' } }],
  exports: [DatabaseService],
})
export class DatabaseModule {} // host is baked in; not reusable
```

**Correct:**

```typescript
@Module({})
export class DatabaseModule {
  static forRoot(options: DatabaseOptions): DynamicModule {
    return {
      module: DatabaseModule,
      providers: [
        { provide: 'DB_OPTIONS', useValue: options },
        DatabaseService,
      ],
      exports: [DatabaseService],
    };
  }
}

// imports: [DatabaseModule.forRoot({ host: process.env.DB_HOST })]
```

Use `forRootAsync({ useFactory, inject })` when options depend on other providers. Reference: https://docs.nestjs.com/fundamentals/dynamic-modules
