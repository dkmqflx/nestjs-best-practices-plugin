---
title: Return a Consistent Error Shape
impact: MEDIUM
tags: response-shape,json,api-contract,timestamp,path
---

## Return a Consistent Error Shape

Clients parse error responses programmatically, so the body must be predictable. NestJS's default `HttpException` body is `{ statusCode, message, error }`, but unhandled errors and ad-hoc responses drift from that. Standardize every error in your global filter into one shape — for example `statusCode`, `message`, `error`, `timestamp`, and `path` — so every endpoint returns the same contract regardless of which exception was thrown.

**Incorrect:**

```typescript
// Different endpoints invent different bodies
res.status(404).json({ error: 'not found' });          // one place
res.status(400).json({ msg: 'bad', ok: false });        // another place
res.status(500).send('Something broke');                // yet another
```

**Correct:**

```typescript
catch(exception: unknown, host: ArgumentsHost) {
  const ctx = host.switchToHttp();
  const request = ctx.getRequest();
  const response = ctx.getResponse();

  const status =
    exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

  response.status(status).json({
    statusCode: status,
    message:
      exception instanceof HttpException ? exception.message : 'Internal server error',
    error: HttpStatus[status],
    timestamp: new Date().toISOString(),
    path: request.url,
  });
}
```
