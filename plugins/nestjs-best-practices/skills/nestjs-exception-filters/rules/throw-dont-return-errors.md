---
title: Throw Exceptions, Don't Hand-Craft Error Responses
impact: HIGH
tags: throw,filters,separation-of-concerns,controllers,response
---

## Throw Exceptions, Don't Hand-Craft Error Responses

Injecting the raw response object (`@Res()`) and manually setting the status/body bypasses the entire exceptions layer — filters, interceptors, and consistent formatting no longer apply, and the handler becomes coupled to Express/Fastify. Instead, `throw` an exception and let your global filter format it. This keeps handlers focused on the happy path and routes every error through one place.

**Incorrect:**

```typescript
import { Controller, Get, Param, Res } from '@nestjs/common';
import { Response } from 'express';

@Controller('users')
export class UsersController {
  @Get(':id')
  async findOne(@Param('id') id: string, @Res() res: Response) {
    const user = await this.service.find(id);
    if (!user) {
      // Bypasses filters; framework-coupled; inconsistent shape
      return res.status(404).json({ error: 'not found' });
    }
    return res.json(user);
  }
}
```

**Correct:**

```typescript
import { Controller, Get, Param, NotFoundException } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get(':id')
  async findOne(@Param('id') id: string) {
    const user = await this.service.find(id);
    if (!user) {
      throw new NotFoundException(`User ${id} not found`); // filter formats it
    }
    return user; // Nest serializes the success response
  }
}
```
