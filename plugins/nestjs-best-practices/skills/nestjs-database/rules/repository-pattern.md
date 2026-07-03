---
title: Inject Repositories into Providers; Keep DB Access out of Controllers
impact: HIGH
tags: typeorm, repository, injectrepository, layering, controllers
---

## Inject Repositories into Providers; Keep DB Access out of Controllers

Controllers should handle HTTP concerns (routing, validation, status codes) and delegate persistence to services. Register an entity's repository with `TypeOrmModule.forFeature([Entity])`, then inject it into a service with `@InjectRepository`. This keeps data access testable and reusable, and prevents query logic from leaking into the transport layer.

**Incorrect:**

```typescript
// Controller talks to the database directly
@Controller('users')
export class UsersController {
  constructor(
    @InjectRepository(User) private readonly repo: Repository<User>,
  ) {}

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.repo.findOneBy({ id }); // persistence in the controller
  }
}
```

**Correct:**

```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User) private readonly users: Repository<User>,
  ) {}

  findOne(id: string) {
    return this.users.findOneBy({ id });
  }
}

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }
}

// users.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

> Prisma/Mongoose: inject `PrismaService` or `@InjectModel(User.name) Model<User>` into services, never into controllers.

Reference: https://docs.nestjs.com/techniques/database
