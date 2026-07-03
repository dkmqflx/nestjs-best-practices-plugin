---
title: Validate Nested Objects and Arrays
impact: HIGH
tags: validation,nested,ValidateNested,Type,class-transformer,arrays
---

## Validate Nested Objects and Arrays

Validation does not recurse into nested objects automatically. Without `@ValidateNested` and a `@Type` hint, nested payloads stay plain objects and their decorators never run. Use `each: true` for arrays of DTOs.

**Incorrect:**

```typescript
export class CreateOrderDto {
  // address is never validated as an AddressDto
  address: AddressDto;
  items: ItemDto[];
}
```

**Correct:**

```typescript
import { ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';

export class CreateOrderDto {
  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;

  @ValidateNested({ each: true })
  @Type(() => ItemDto)
  items: ItemDto[];
}
```
