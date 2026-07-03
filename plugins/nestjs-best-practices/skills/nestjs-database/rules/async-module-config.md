---
title: Configure the Data Layer with forRootAsync + ConfigService
impact: CRITICAL
tags: typeorm, prisma, mongoose, config, security, credentials
---

## Configure the Data Layer with forRootAsync + ConfigService

Never hardcode database credentials in source. Load connection settings from environment via `@nestjs/config` and wire them with `forRootAsync`/`useFactory`, which lets the connection wait for `ConfigModule` and keeps secrets out of the repo. `synchronize` and `logging` should also be config-driven so production never auto-mutates schema (see `migrations-not-synchronize`).

**Incorrect:**

```typescript
// Hardcoded secrets committed to the repo
TypeOrmModule.forRoot({
  type: 'postgres',
  host: 'prod-db.internal',
  username: 'admin',
  password: 'S3cr3t!',     // leaked credentials
  database: 'app',
  synchronize: true,        // dangerous in production
  entities: [User],
});
```

**Correct:**

```typescript
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    TypeOrmModule.forRootAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        type: 'postgres',
        host: config.get<string>('DB_HOST'),
        port: config.get<number>('DB_PORT'),
        username: config.get<string>('DB_USER'),
        password: config.get<string>('DB_PASSWORD'),
        database: config.get<string>('DB_NAME'),
        autoLoadEntities: true,
        synchronize: config.get('NODE_ENV') !== 'production',
      }),
    }),
  ],
})
export class AppModule {}
```

> Prisma: keep the URL in `DATABASE_URL` (read from `.env` by Prisma) and inject a `PrismaService`. Mongoose: use `MongooseModule.forRootAsync({ useFactory })` reading `MONGODB_URI` from `ConfigService`.

Reference: https://docs.nestjs.com/techniques/database
