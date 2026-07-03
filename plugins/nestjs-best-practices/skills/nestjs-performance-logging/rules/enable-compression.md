---
title: Enable response compression
impact: MEDIUM
tags: performance,compression,gzip,bandwidth
---

## Enable response compression

Compressing response bodies (gzip/brotli) shrinks payloads dramatically for JSON/text APIs, lowering bandwidth cost and time-to-last-byte. Use the adapter-native plugin: `@fastify/compress` for Fastify, the `compression` middleware for Express. Skip compression for very small payloads, where the CPU cost outweighs the savings.

**Incorrect:**

```typescript
// No compression — full-size JSON over the wire
const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  new FastifyAdapter(),
);
await app.listen(3000);
```

**Correct:**

```typescript
import compression from '@fastify/compress';

const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  new FastifyAdapter(),
);
await app.register(compression); // Express: app.use(compression())
await app.listen(3000);
```
