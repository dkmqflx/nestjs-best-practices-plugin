---
title: Group related config with registerAs() namespaces and typed access
impact: MEDIUM
tags: registeras, configtype, namespace, load
---

## Group related config with registerAs() namespaces and typed access

For related settings (database, redis, jwt), define a namespace with `registerAs()` and
load it via `load: [...]`. You then get strongly typed access by injecting
`ConfigType<typeof config>` with `@Inject(config.KEY)`, instead of fishing individual flat
keys out of `ConfigService` and remembering their shapes.

**Incorrect:**

```typescript
// Flat, ungrouped keys read one-by-one everywhere
const host = config.get<string>('DATABASE_HOST');
const port = config.get<number>('DATABASE_PORT');
const user = config.get<string>('DATABASE_USER');
```

**Correct:**

```typescript
// database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST,
  port: parseInt(process.env.DATABASE_PORT ?? '5432', 10),
  user: process.env.DATABASE_USER,
}));

// app.module.ts
ConfigModule.forRoot({ isGlobal: true, load: [databaseConfig] });

// service — fully typed, grouped object
@Injectable()
export class DatabaseService {
  constructor(
    @Inject(databaseConfig.KEY)
    private readonly dbConfig: ConfigType<typeof databaseConfig>,
  ) {}

  connect() {
    return createPool({ host: this.dbConfig.host, port: this.dbConfig.port });
  }
}
```
