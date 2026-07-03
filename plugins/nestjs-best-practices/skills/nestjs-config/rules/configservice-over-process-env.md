---
title: Inject ConfigService instead of reading process.env directly
impact: HIGH
tags: configservice, process-env, di, testability
---

## Inject ConfigService instead of reading process.env directly

Reading `process.env` scattered through the codebase bypasses validation, defaults, and
type coercion, and makes code hard to test (you must mutate global state). Inject
`ConfigService` and read config through `get()` — it returns validated, parsed values and
is trivially mockable in tests.

**Incorrect:**

```typescript
@Injectable()
export class MailService {
  send() {
    const host = process.env.SMTP_HOST; // unvalidated, untyped, hard to mock
    const port = Number(process.env.SMTP_PORT);
    // ...
  }
}
```

**Correct:**

```typescript
@Injectable()
export class MailService {
  constructor(private readonly configService: ConfigService) {}

  send() {
    const host = this.configService.get<string>('SMTP_HOST');
    const port = this.configService.get<number>('SMTP_PORT');
    // ...
  }
}
```
