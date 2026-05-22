## development-workflow

> Development workflow, common tasks, and adding new features to the users service


# Users Service - Development Workflow

## Adding a New Feature

Follow these steps to add new functionality while maintaining hexagonal architecture:

### 1. Define Domain Entity (if needed)

**Location**: `src/application/core/domain/`

If your feature requires a new entity or modifying existing ones:

```typescript
// src/application/core/domain/user.entity.ts
export class User {
  id: string;
  email: string;
  password?: string;
  name: string;
  city: string;
  uf: string;
  zipcode?: string;
  personType: string;
  avatarPath?: string;
}
```

Remember: Domain entities must be framework-agnostic!

### 2. Define Repository Port (if needed)

**Location**: `src/application/ports/out/`

Create or update the repository interface:

```typescript
// src/application/ports/out/users-repository.port.ts
export interface UserRepositoryPort {
  // Existing methods...

  // New method
  updateAvatar(id: string, avatarPath: string): Promise<User | null>;
}
```

### 3. Create Use Case

**Location**: `src/application/ports/in/`

Implement the business logic:

```typescript
// src/application/ports/in/user/updateUserAvatar.useCase.ts
import { Injectable, Inject } from '@nestjs/common';
import { UseCase } from '@/application/types/useCase.types';
import { Result, ResultFactory } from '@/application/types/result.types';
import { User } from '@/application/core/domain/user.entity';
import { UserRepositoryPort } from '@/application/ports/out/users-repository.port';
import { USERS_REPOSITORY } from '@/constants';
import { ErrorsEnum } from '@/application/core/errors/errors.enum';

export interface UpdateUserAvatarInput {
  userId: string;
  avatarPath: string;
}

@Injectable()
export class UpdateUserAvatarUseCase
  implements UseCase<UpdateUserAvatarInput, Promise<Result<User>>>
{
  constructor(
    @Inject(USERS_REPOSITORY)
    private readonly usersRepository: UserRepositoryPort,
  ) {}

  async execute(input: UpdateUserAvatarInput): Promise<Result<User>> {
    const user = await this.usersRepository.findById(input.userId);

    if (!user) {
      return ResultFactory.failure(ErrorsEnum.UserNotFound);
    }

    const updatedUser = await this.usersRepository.updateAvatar(
      input.userId,
      input.avatarPath,
    );

    if (!updatedUser) {
      return ResultFactory.failure(ErrorsEnum.UserNotFound);
    }

    return ResultFactory.success(updatedUser);
  }
}
```

### 4. Update Service

**Location**: `src/application/core/service/`

Add the new use case to the service:

```typescript
// src/application/core/service/users.service.ts
@Injectable()
export class UsersService {
  constructor(
    private readonly createUserUseCase: CreateUserUseCase,
    private readonly changePasswordUseCase: ChangePasswordUseCase,
    private readonly updateUserAvatarUseCase: UpdateUserAvatarUseCase, // New
  ) {}

  // Existing methods...

  async uploadAvatar(
    userId: string,
    avatarPath: string,
  ): Promise<Result<User>> {
    return this.updateUserAvatarUseCase.execute({ userId, avatarPath });
  }
}
```

### 5. Implement Repository

**Location**: `src/adapters/out/`

Implement the port interface:

```typescript
// src/adapters/out/users.repository.ts
@Injectable()
export class UsersRepository implements UserRepositoryPort {
  constructor(
    @InjectRepository(Users)
    private readonly usersRepository: Repository<Users>,
  ) {}

  // Existing methods...

  async updateAvatar(id: string, avatarPath: string): Promise<User | null> {
    const user = await this.usersRepository.findOneBy({ id });

    if (!user) {
      return null;
    }

    user.avatarPath = avatarPath;
    const savedUser = await this.usersRepository.save(user);

    return UserMapper.toDomain(savedUser);
  }
}
```

### 6. Add Controller Endpoint

**Location**: `src/adapters/in/`

Expose via HTTP:

```typescript
// src/adapters/in/user.controller.ts
@Controller('/users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  // Existing endpoints...

  @Post(':id/avatar')
  @UseInterceptors(FileInterceptor('avatar', uploadConfig))
  async uploadAvatar(
    @Param('id') id: string,
    @UploadedFile() file: Express.Multer.File,
  ) {
    const avatarPath = `temp/uploads/${file.filename}`;
    const result = await this.usersService.uploadAvatar(id, avatarPath);

    if (!result.isSuccess) {
      switch (result.error) {
        case ErrorsEnum.UserNotFound:
          throw new HttpException(result.error, HttpStatus.NOT_FOUND);
        default:
          throw new HttpException(result.error, HttpStatus.BAD_REQUEST);
      }
    }

    return result.value;
  }
}
```

### 7. Update Module

**Location**: `src/user.module.ts`

Register the new use case:

```typescript
@Module({
  imports: [
    /* ... */
  ],
  controllers: [UsersController],
  providers: [
    { provide: USERS_REPOSITORY, useClass: UsersRepository },
    { provide: DONOR_REPOSITORY, useClass: DonorRepository },
    { provide: COMPANY_REPOSITORY, useClass: CompanyRepository },
    CreateUserUseCase,
    ChangePasswordUseCase,
    UpdateUserAvatarUseCase, // Add this
    CreateDonorUseCase,
    CreateCompanyUseCase,
    UsersService,
  ],
})
export class AppModule {}
```

### 8. Write Tests

Create tests for each layer:

```typescript
// src/application/ports/in/user/__tests__/updateUserAvatar.useCase.spec.ts
describe('UpdateUserAvatarUseCase', () => {
  let useCase: UpdateUserAvatarUseCase;
  let mockRepository: UserRepositoryPort;

  beforeEach(() => {
    mockRepository = {
      findById: jest.fn(),
      updateAvatar: jest.fn(),
    } as any;

    useCase = new UpdateUserAvatarUseCase(mockRepository);
  });

  it('should update user avatar successfully', async () => {
    const user = { id: '1', name: 'Test', avatarPath: null };
    const updatedUser = { ...user, avatarPath: 'temp/uploads/avatar.jpg' };

    jest.spyOn(mockRepository, 'findById').mockResolvedValue(user);
    jest.spyOn(mockRepository, 'updateAvatar').mockResolvedValue(updatedUser);

    const result = await useCase.execute({
      userId: '1',
      avatarPath: 'temp/uploads/avatar.jpg',
    });

    expect(result.isSuccess).toBe(true);
    expect(result.value.avatarPath).toBe('temp/uploads/avatar.jpg');
  });

  it('should fail if user not found', async () => {
    jest.spyOn(mockRepository, 'findById').mockResolvedValue(null);

    const result = await useCase.execute({
      userId: '999',
      avatarPath: 'temp/uploads/avatar.jpg',
    });

    expect(result.isSuccess).toBe(false);
    expect(result.error).toBe(ErrorsEnum.UserNotFound);
  });
});
```

## Common Development Tasks

### Running the Application

```bash
# Development mode (with hot reload)
npm run start:dev

# Debug mode
npm run start:debug

# Production mode
npm run start:prod
```

### Running with Docker

```bash
# Start all services
docker-compose up

# Start in detached mode
docker-compose up -d

# View logs
docker-compose logs -f users-service

# Stop services
docker-compose down
```

### Database Operations

```bash
# Connect to PostgreSQL
docker exec -it <postgres-container-name> psql -U <username> -d <database>

# List tables
\dt

# Query users
SELECT * FROM users;

# Query donors
SELECT * FROM donors;

# Query companies
SELECT * FROM companies;
```

### Testing

```bash
# Run all tests
npm test

# Watch mode
npm run test:watch

# Coverage
npm run test:cov

# E2E tests
npm run test:e2e
```

### Linting and Formatting

```bash
# Run linter
npm run lint

# Format code
npm run format

# Build
npm run build
```

## Code Review Checklist

Before submitting changes, verify:

- [ ] Domain entities have no framework dependencies
- [ ] Use cases return `Result<T>` type
- [ ] Controllers only handle HTTP concerns
- [ ] Repository ports define interfaces, not implementations
- [ ] New providers added to module
- [ ] All dependencies injected via constructor
- [ ] `@Injectable()` decorator applied to services
- [ ] Proper error handling (Result pattern for business, exceptions for technical)
- [ ] Tests written for new functionality
- [ ] Code formatted (`npm run format`)
- [ ] No linter errors (`npm run lint`)
- [ ] Imports organized correctly
- [ ] No circular dependencies
- [ ] Documentation updated if needed

## Debugging

### Using NestJS Logger

```typescript
import { Logger } from '@nestjs/common';

@Injectable()
export class UsersService {
  private readonly logger = new Logger(UsersService.name);

  async createUser(user: CreateUserRequest): Promise<Result<User>> {
    this.logger.debug('Creating user', { user });
    const result = await this.createUserUseCase.execute(user);
    this.logger.debug('User created', { result });
    return result;
  }
}
```

### Debug Mode with VSCode

Add to `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach to NestJS",
      "port": 9229,
      "restart": true,
      "sourceMaps": true,
      "outFiles": ["${workspaceFolder}/dist/**/*.js"]
    }
  ]
}
```

Then run:

```bash
npm run start:debug
```

## Environment Variables

### Development (.env)

```env
PORT=3000
POSTGRES_HOST=localhost
POSTGRES_USERNAME=postgres
POSTGRES_PASSWORD=password
POSTGRES_DATABASE=users_db
JWT_SECRET=your-secret-key
NODE_ENV=development
```

### Production

Ensure these are set in your deployment environment:

- `PORT`
- `POSTGRES_HOST`
- `POSTGRES_USERNAME`
- `POSTGRES_PASSWORD`
- `POSTGRES_DATABASE`
- `JWT_SECRET`
- `NODE_ENV=production`

## Microservice Communication

Since this is part of the Sangue Solidario project with multiple microservices:

### Publishing Events (when needed)

1. Create event types in domain
2. Use message broker (RabbitMQ, Kafka, etc.)
3. Keep events in `src/application/core/events/`
4. Publish from use cases after successful operations

Example:

```typescript
export interface UserCreatedEvent {
  userId: string;
  email: string;
  personType: PersonType;
  timestamp: Date;
}
```

### Consuming Events

1. Create event handlers in `src/adapters/in/events/`
2. Map events to use case calls
3. Handle idempotency

## Monitoring and Health Checks

### Health Check Endpoint

Add NestJS Terminus for health checks:

```bash
npm install @nestjs/terminus
```

```typescript
// src/adapters/in/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  TypeOrmHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([() => this.db.pingCheck('database')]);
  }
}
```

## Performance Tips

1. **Use indexes** on frequently queried fields (email, cpf, cnpj)
2. **Limit query results** with pagination
3. **Use projection** to fetch only needed fields
4. **Cache frequent queries** (Redis, etc.)
5. **Monitor slow queries** in PostgreSQL
6. **Use connection pooling** (already configured in TypeORM)

## Common Issues and Solutions

### Issue: "Cannot inject repository"

**Solution**: Ensure provider is registered in module with correct token:

```typescript
{ provide: USERS_REPOSITORY, useClass: UsersRepository }
```

### Issue: "Circular dependency"

**Solution**:

- Services should not inject use cases that depend on them
- Use forwardRef() if unavoidable (but reconsider design)

### Issue: "Domain entity has framework dependency"

**Solution**:

- Keep domain entities pure
- Create separate adapter entities for TypeORM
- Map between domain and adapter entities in repositories and mappers

### Issue: "Tests fail with DI errors"

**Solution**:

- Provide all dependencies in test module
- Use mocks for external dependencies
- Import required NestJS testing utilities

## API Documentation

Consider adding Swagger/OpenAPI:

```bash
npm install @nestjs/swagger swagger-ui-express
```

```typescript
// src/main.ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('Users Service')
    .setDescription('User management and authentication API')
    .setVersion('1.0')
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api-docs', app, document);

  await app.listen(process.env.PORT ?? 3000);
}
```

## Git Workflow

### Branch Naming

- `feature/` - New features
- `fix/` - Bug fixes
- `refactor/` - Code refactoring
- `docs/` - Documentation

Example: `feature/add-avatar-upload`

### Commit Messages

Follow conventional commits:

- `feat: add avatar upload endpoint`
- `fix: correct password hashing validation`
- `refactor: simplify use case structure`
- `docs: update API documentation`
- `test: add unit tests for user creation`

## Migration Strategy

When changing domain entities:

1. **Add new field** (non-breaking)
   - Add to domain entity as optional
   - Update use cases to handle both old and new
   - Run database migration
   - Make required if needed

2. **Remove field** (breaking)
   - Deprecate first
   - Update all use cases
   - Run database migration
   - Remove from entity

3. **Rename field** (breaking)
   - Add new field
   - Dual-write period
   - Run data migration
   - Remove old field

Example TypeORM migration:

```typescript
import { MigrationInterface, QueryRunner, TableColumn } from 'typeorm';

export class AddAvatarPathToUsers1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.addColumn(
      'users',
      new TableColumn({
        name: 'avatar_path',
        type: 'varchar',
        isNullable: true,
      }),
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('users', 'avatar_path');
  }
}
```

## Resources

### NestJS Documentation

https://docs.nestjs.com

### Hexagonal Architecture

- Ports and Adapters pattern
- Clean Architecture principles
- Domain-Driven Design (DDD)

### PostgreSQL & TypeORM

https://typeorm.io

### TypeScript Best Practices

https://www.typescriptlang.org/docs/

---
> Source: [c3ny/users-service](https://github.com/c3ny/users-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
