---
title: Propagate correlation IDs
impact: MEDIUM
tags: observability,logging,tracing,correlation
---

## Propagate correlation IDs

A correlation (request) ID stitches every log line of a single request together and follows the request across service boundaries, making incident debugging tractable. Reuse an inbound `X-Request-Id` header if present, otherwise generate one, and bind it to every log line automatically. With `nestjs-pino` this is configuration, not manual plumbing.

**Incorrect:**

```typescript
// each log line stands alone — no way to group a request's logs
this.logger.info('fetching user');
this.logger.info('user not found');
```

**Correct:**

```typescript
import { randomUUID } from 'crypto';

LoggerModule.forRoot({
  pinoHttp: {
    genReqId: (req) => req.headers['x-request-id'] ?? randomUUID(),
    // pino-http auto-attaches reqId to every log line in the request
  },
});
```
