---
title: Disable synchronize in Production; Manage Schema with Migrations
impact: CRITICAL
tags: typeorm, migrations, synchronize, schema, production, data-loss
---

## Disable synchronize in Production; Manage Schema with Migrations

`synchronize: true` makes TypeORM alter the database to match entities on every boot — convenient locally but capable of dropping columns and destroying data in production. Set `synchronize: false` everywhere but development, and evolve the schema with versioned migrations that are reviewed and run deliberately.

**Incorrect:**

```typescript
// Schema auto-mutates on every deploy — silent data loss
TypeOrmModule.forRoot({
  type: 'postgres',
  entities: [User, Order],
  synchronize: true, // never in production
});
```

**Correct:**

```typescript
// data-source.ts (used by the TypeORM CLI and the app)
export const AppDataSource = new DataSource({
  type: 'postgres',
  url: process.env.DATABASE_URL,
  entities: ['dist/**/*.entity.js'],
  migrations: ['dist/migrations/*.js'],
  synchronize: false,
  migrationsRun: true, // apply pending migrations on startup
});

// Generate & run via CLI:
//   typeorm migration:generate src/migrations/AddUserEmail -d data-source.ts
//   typeorm migration:run -d data-source.ts
```

> Prisma: never use `prisma db push` against production — use `prisma migrate deploy` to apply committed migrations. Mongoose has no fixed schema, but still script/version data changes rather than mutating ad hoc.

Reference: https://docs.nestjs.com/techniques/database
