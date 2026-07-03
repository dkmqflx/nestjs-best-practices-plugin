---
title: Cap Request Body Size
impact: MEDIUM
tags: dos,body-size,payload,express,fastify
---

## Cap Request Body Size

An unbounded request body lets a single client exhaust memory or CPU by POSTing a multi-megabyte JSON blob — a cheap denial-of-service. Express's body parser defaults to a 100kb limit, but apps often raise it for file uploads and forget to scope it. Set an explicit, conservative limit that matches your real payloads. On Express, configure the body parser limit; on Fastify, set `bodyLimit` at adapter creation.

**Incorrect:**

```typescript
// Raised globally for one upload route, leaving every endpoint exposed
import * as bodyParser from 'body-parser';
app.use(bodyParser.json({ limit: '50mb' }));
```

**Correct:**

```typescript
// main.ts (Express) — explicit, conservative global limit
import * as bodyParser from 'body-parser';

app.use(bodyParser.json({ limit: '100kb' }));
app.use(bodyParser.urlencoded({ limit: '100kb', extended: true }));
// Apply larger limits only to specific upload routes, not globally.

// main.ts (Fastify) — set bodyLimit on the adapter
// const app = await NestFactory.create<NestFastifyApplication>(
//   AppModule,
//   new FastifyAdapter({ bodyLimit: 100 * 1024 }),
// );
```
