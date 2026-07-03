---
title: Expose health checks with Terminus
impact: HIGH
tags: observability,health,terminus,kubernetes
---

## Expose health checks with Terminus

Orchestrators need a cheap endpoint to decide if a pod is alive (liveness) and ready for traffic (readiness). `@nestjs/terminus` provides composable indicators (DB, HTTP, disk, memory) behind a `HealthCheckService`. Keep liveness trivial and put dependency checks (DB, downstream services) in readiness so a slow dependency drains traffic instead of restarting the pod.

**Incorrect:**

```typescript
@Get('/health')
health() {
  return { status: 'ok' }; // lies even when the DB is down
}
```

**Correct:**

```typescript
import { HealthCheck, HealthCheckService, TypeOrmHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
  ) {}

  @Get('readiness')
  @HealthCheck()
  readiness() {
    return this.health.check([() => this.db.pingCheck('database')]);
  }
}
```
