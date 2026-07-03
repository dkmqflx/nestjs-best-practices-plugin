---
title: Use Built-in HTTP Exceptions
impact: HIGH
tags: http-exception,status-code,controllers,services
---

## Use Built-in HTTP Exceptions

NestJS provides standard exceptions (`BadRequestException`, `NotFoundException`, `UnauthorizedException`, `ForbiddenException`, `ConflictException`, etc.) that all extend `HttpException` and carry the correct HTTP status. Throwing a plain `Error` bypasses this — the default filter can't infer a status, so the client gets a generic `500 Internal Server Error` even when the real problem is a bad request or a missing record.

**Incorrect:**

```typescript
@Injectable()
export class UsersService {
  async findOne(id: string) {
    const user = await this.repo.findById(id);
    if (!user) {
      // Generic Error -> client sees 500, not 404
      throw new Error(`User ${id} not found`);
    }
    return user;
  }
}
```

**Correct:**

```typescript
import { NotFoundException } from '@nestjs/common';

@Injectable()
export class UsersService {
  async findOne(id: string) {
    const user = await this.repo.findById(id);
    if (!user) {
      throw new NotFoundException(`User ${id} not found`); // -> 404
    }
    return user;
  }
}
```
