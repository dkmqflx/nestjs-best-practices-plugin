---
title: Keep Controllers Thin
impact: CRITICAL
tags: controllers,services,separation-of-concerns,architecture
---

## Keep Controllers Thin

A controller's job is to map an HTTP request to a handler, extract/validate input, and return a result. Business logic, persistence, and orchestration belong in injectable services so they can be unit-tested and reused outside the HTTP layer.

**Incorrect:**

```typescript
@Controller('users')
export class UsersController {
  constructor(@InjectRepository(User) private readonly repo: Repository<User>) {}

  @Post()
  async create(@Body() dto: CreateUserDto) {
    const existing = await this.repo.findOne({ where: { email: dto.email } });
    if (existing) throw new ConflictException('Email taken');
    const hashed = await bcrypt.hash(dto.password, 10);
    return this.repo.save(this.repo.create({ ...dto, password: hashed }));
  }
}
```

**Correct:**

```typescript
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  create(@Body() dto: CreateUserDto): Promise<User> {
    return this.usersService.create(dto);
  }
}
```
