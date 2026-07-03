---
title: Don't log sensitive data
impact: CRITICAL
tags: security,logging,pii,redaction
---

## Don't log sensitive data

Logging raw request bodies, headers, or error objects routinely leaks passwords, tokens, API keys, and PII into log stores where they persist and widen breach impact. Never log whole `req`/`dto`/`headers` blobs. Configure redaction at the logger so sensitive paths are stripped before serialization — pino has built-in `redact` support.

**Incorrect:**

```typescript
this.logger.info({ body: req.body, headers: req.headers }, 'login attempt');
// leaks password, Authorization header, cookies
```

**Correct:**

```typescript
LoggerModule.forRoot({
  pinoHttp: {
    redact: {
      paths: ['req.headers.authorization', 'req.headers.cookie', 'req.body.password', '*.token'],
      censor: '[REDACTED]',
    },
  },
});

// log only the fields you need
this.logger.info({ userId }, 'login attempt');
```
