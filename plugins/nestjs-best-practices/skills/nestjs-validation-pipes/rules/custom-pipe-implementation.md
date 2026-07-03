---
title: Build Reusable Custom Pipes with PipeTransform
impact: MEDIUM
tags: pipes,PipeTransform,custom,transform,validation
---

## Build Reusable Custom Pipes with PipeTransform

When a built-in pipe does not cover your validation or transformation, implement `PipeTransform` instead of repeating ad-hoc checks inside handlers. Return the (optionally transformed) value or throw an `HttpException` to reject. The pipe is then reusable and unit-testable.

**Incorrect:**

```typescript
@Get()
list(@Query('sort') sort: string) {
  // inline, duplicated across every handler that takes `sort`
  if (sort !== 'asc' && sort !== 'desc') {
    throw new BadRequestException('Invalid sort');
  }
}
```

**Correct:**

```typescript
@Injectable()
export class SortPipe implements PipeTransform<string, 'asc' | 'desc'> {
  transform(value: string, _metadata: ArgumentMetadata): 'asc' | 'desc' {
    if (value !== 'asc' && value !== 'desc') {
      throw new BadRequestException('sort must be "asc" or "desc"');
    }
    return value;
  }
}

@Get()
list(@Query('sort', SortPipe) sort: 'asc' | 'desc') {}
```
