---
title: Use Custom Provider Tokens to Inject Abstractions
impact: MEDIUM
tags: custom-providers, tokens, useFactory, useValue, abstraction
---

## Use Custom Provider Tokens to Inject Abstractions

Beyond standard class providers, NestJS supports `useClass`, `useValue`, `useFactory`, and `useExisting`. Bind an injection token (string, symbol, or abstract class) to an implementation so consumers depend on a contract rather than a concrete class — easing testing and swapping implementations. Because TypeScript interfaces vanish at runtime, use a token plus `@Inject()`.

**Incorrect:**

```typescript
// Consumer is hard-wired to a concrete class; hard to mock or swap
@Injectable()
export class NotificationService {
  constructor(private readonly mailer: SmtpMailer) {}
}
```

**Correct:**

```typescript
export const MAILER = Symbol('MAILER');
export interface Mailer { send(to: string, body: string): Promise<void>; }

@Module({
  providers: [
    { provide: MAILER, useClass: SmtpMailer },
    // or useValue / useFactory:
    // { provide: MAILER, useFactory: (c: ConfigService) =>
    //     new SmtpMailer(c.get('SMTP_URL')), inject: [ConfigService] },
  ],
  exports: [MAILER],
})
export class MailModule {}

@Injectable()
export class NotificationService {
  constructor(@Inject(MAILER) private readonly mailer: Mailer) {}
}
```

Reference: https://docs.nestjs.com/fundamentals/custom-providers
