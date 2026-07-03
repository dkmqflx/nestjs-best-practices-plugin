---
title: Use structured JSON logging
impact: HIGH
tags: logging,observability,pino,json
---

## Use structured JSON logging

`console.log` produces unstructured text that log platforms can't filter, search, or aggregate. Use `nestjs-pino` to emit structured JSON with consistent fields and high throughput, and wire it as the app logger so framework logs flow through it too. JSON logs are essential for centralized logging on ECS/Kubernetes/CloudWatch.

**Incorrect:**

```typescript
@Injectable()
export class OrdersService {
  create(dto: CreateOrderDto) {
    console.log('creating order for ' + dto.userId); // unsearchable text
  }
}
```

**Correct:**

```typescript
// app.module.ts
import { LoggerModule } from 'nestjs-pino';

@Module({ imports: [LoggerModule.forRoot()] })
export class AppModule {}

// main.ts
import { Logger } from 'nestjs-pino';
const app = await NestFactory.create(AppModule, { bufferLogs: true });
app.useLogger(app.get(Logger));

// orders.service.ts
import { PinoLogger } from 'nestjs-pino';
constructor(private readonly logger: PinoLogger) {}
create(dto: CreateOrderDto) {
  this.logger.info({ userId: dto.userId }, 'creating order');
}
```
