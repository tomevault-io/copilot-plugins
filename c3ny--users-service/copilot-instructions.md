## architecture

> This microservice follows **Hexagonal Architecture** (Ports and Adapters), which promotes:


# Users Service - Hexagonal Architecture Overview

## Architecture Pattern

This microservice follows **Hexagonal Architecture** (Ports and Adapters), which promotes:

- Clear separation of concerns
- Business logic independence from frameworks and external dependencies
- Testability and maintainability
- Dependency inversion principle

## Project Structure

### Core Layers

#### 1. Application Core (`src/application/core/`)

The **innermost layer** containing pure business logic with NO external dependencies.

**Domain Entities** (`src/application/core/domain/`)

- Pure TypeScript classes/interfaces representing business concepts
- Examples: [user.entity.ts](mdc:src/application/core/domain/user.entity.ts), [donor.entity.ts](mdc:src/application/core/domain/donor.entity.ts), [company.entity.ts](mdc:src/application/core/domain/company.entity.ts)
- Must NOT import from adapters, NestJS, or any framework
- Should only use native TypeScript types and other domain entities

**Services** (`src/application/core/service/`)

- Orchestrates use cases
- Example: [users.service.ts](mdc:src/application/core/service/users.service.ts)
- Depends only on use case interfaces (ports/in)
- Must NOT contain business logic (delegate to use cases)

**Errors** (`src/application/core/errors/`)

- Domain-specific error definitions
- Example: [errors.enum.ts](mdc:src/application/core/errors/errors.enum.ts)

#### 2. Application Ports (`src/application/ports/`)

Define **interfaces** for communication between layers.

**Input Ports** (`src/application/ports/in/`)

- Define use case interfaces
- Examples: [createUser.useCase.ts](mdc:src/application/ports/in/user/createUser.useCase.ts), [changePassword.useCase.ts](mdc:src/application/ports/in/user/changePassword.useCase.ts), [updateUserAvatar.useCase.ts](mdc:src/application/ports/in/user/updateUserAvatar.useCase.ts)
- Implement the `UseCase<T, R>` interface from [useCase.types.ts](mdc:src/types/useCase.types.ts)
- Return `Result<T>` type for consistent error handling
- Can inject output ports (repositories) via dependency injection

**Output Ports** (`src/application/ports/out/`)

- Define repository/external service interfaces
- Examples: [users-repository.port.ts](mdc:src/application/ports/out/users-repository.port.ts), [donor-repository.port.ts](mdc:src/application/ports/out/donor-repository.port.ts)
- Abstractions that adapters must implement
- Use domain entities, NOT framework-specific types

#### 3. Adapters (`src/adapters/`)

Connect the application to the outside world.

**Input Adapters** (`src/adapters/in/`)

- HTTP controllers, event listeners, CLI handlers
- Example: [user.controller.ts](mdc:src/adapters/in/user.controller.ts)
- Use NestJS decorators (`@Controller`, `@Post`, etc.)
- Translate HTTP requests to use case calls
- Depend on services from `application/core/service/`

**Output Adapters** (`src/adapters/out/`)

- Repository implementations, external API clients
- Examples: [users.repository.ts](mdc:src/adapters/out/users.repository.ts), [donor.repository.ts](mdc:src/adapters/out/donor.repository.ts)
- Implement port interfaces from `application/ports/out/`
- Handle framework-specific logic (TypeORM, HTTP clients, etc.)

**Adapter Domain Models** (`src/adapters/out/domain/`)

- Framework-specific entity mappings (e.g., TypeORM entities)
- Examples: [user.entity.ts](mdc:src/adapters/out/domain/user.entity.ts), [donor.entity.ts](mdc:src/adapters/out/domain/donor.entity.ts)
- Map between domain entities and persistence models

**Mappers** (`src/adapters/out/mappers/`)

- Transform between domain entities and persistence entities
- Examples: [user.mapper.ts](mdc:src/adapters/out/mappers/user.mapper.ts), [donor.mapper.ts](mdc:src/adapters/out/mappers/donor.mapper.ts)

### Shared Types (`src/types/`)

- Common type definitions used across layers
- [user.types.ts](mdc:src/types/user.types.ts) - User-related types
- [result.types.ts](mdc:src/types/result.types.ts) - Result pattern for error handling
- [useCase.types.ts](mdc:src/types/useCase.types.ts) - Use case interface

### Constants (`src/constants/`)

- Injection tokens and application constants
- Examples: `USERS_REPOSITORY`, `DONOR_REPOSITORY`, `COMPANY_REPOSITORY` tokens for DI

## Dependency Rules

### ✅ Allowed Dependencies

```
Adapters/In → Application/Core/Service → Application/Ports/In → Application/Ports/Out
                                                                          ↑
Adapters/Out ─────────────────────────────────────────────────────────────┘
```

1. **Controllers** (adapters/in) → Services (application/core/service)
2. **Services** → Use Cases (application/ports/in)
3. **Use Cases** → Repository Ports (application/ports/out)
4. **Repositories** (adapters/out) → Repository Ports (implements interface)
5. **Everyone** → Domain Entities (application/core/domain)
6. **Everyone** → Shared Types (src/types)

### ❌ Forbidden Dependencies

1. **Domain entities** MUST NOT import from:
   - NestJS (`@nestjs/*`)
   - TypeORM or any ORM
   - Adapters layer
   - Any framework-specific code

2. **Use Cases** (ports/in) MUST NOT import from:
   - Adapters layer
   - Services (circular dependency)
   - Framework implementations

3. **Repository Ports** (ports/out) MUST NOT import from:
   - Adapters layer
   - Concrete implementations

4. **Application Core** MUST remain framework-agnostic

## Design Patterns

### 1. Result Pattern

Use `Result<T>` type for consistent error handling:

```typescript
// From result.types.ts
type Result<T> = SuccessResult<T> | PartialSuccessResult<T> | FailureResult;

// Usage in use cases
return ResultFactory.success(user);
return ResultFactory.failure(ErrorsEnum.UserNotFound);
```

### 2. Use Case Pattern

Each business operation is a separate use case:

```typescript
@Injectable()
export class CreateUserUseCase implements UseCase<User, Promise<Result<User>>> {
  execute(user: Omit<User, 'id'>): Promise<Result<User>> {
    // Implementation
  }
}
```

### 3. Dependency Injection

- Use NestJS DI container
- Inject interfaces (ports), not implementations
- Define tokens in [constants/index.ts](mdc:src/constants/index.ts)

Example:

```typescript
{ provide: USERS_REPOSITORY, useClass: UsersRepository }
```

### 4. Service Layer as Facade

Services coordinate use cases but don't contain business logic:

```typescript
@Injectable()
export class UsersService {
  constructor(
    private readonly createUserUseCase: CreateUserUseCase,
    private readonly changePasswordUseCase: ChangePasswordUseCase,
    private readonly updateUserAvatarUseCase: UpdateUserAvatarUseCase,
  ) {}

  async createUser(user: CreateUserRequest): Promise<Result<User>> {
    return this.createUserUseCase.execute(user);
  }
}
```

## Module Organization

All dependencies are wired in [user.module.ts](mdc:src/user.module.ts):

1. **Imports**: External modules (TypeOrmModule, HashModule)
2. **Controllers**: Input adapters
3. **Providers**: Services, use cases, and repository implementations

## Naming Conventions

1. **Use Cases**: `{Action}{Entity}UseCase` (e.g., `CreateUserUseCase`, `UpdateUserAvatarUseCase`)
2. **Services**: `{Entity}Service` (e.g., `UsersService`)
3. **Controllers**: `{Entity}Controller` (e.g., `UsersController`)
4. **Repositories**: `{Entity}Repository` (e.g., `UsersRepository`, `DonorRepository`)
5. **Repository Ports**: `{Entity}RepositoryPort` (e.g., `UserRepositoryPort`)
6. **Domain Entities**: `{Entity}` (e.g., `User`, `Donor`, `Company`)
7. **Tokens**: `{ENTITY}_{TYPE}` (e.g., `USERS_REPOSITORY`, `DONOR_REPOSITORY`)

## Best Practices

### When Adding New Features

1. **Start with Domain**: Define entities in `application/core/domain/`
2. **Define Ports**: Create interfaces in `application/ports/`
3. **Implement Use Case**: Add logic in `application/ports/in/`
4. **Create Adapters**: Implement controllers and repositories in `adapters/`
5. **Wire Dependencies**: Update [user.module.ts](mdc:src/user.module.ts)

### Testing Strategy

1. **Unit Tests**: Test use cases in isolation (mock ports)
2. **Integration Tests**: Test adapters with real dependencies
3. **E2E Tests**: Test complete flows through controllers

### Error Handling

1. Use `Result<T>` pattern for business errors
2. Let framework exceptions bubble up for technical errors
3. Define domain errors in [errors.enum.ts](mdc:src/application/core/errors/errors.enum.ts)

### Code Organization

1. One use case per file
2. Keep use cases focused (Single Responsibility Principle)
3. Services should only orchestrate, not implement logic
4. Controllers should be thin (just HTTP concerns)

## Business Domain

### User Management Model

The service manages user accounts and profiles:

- **User Types**: Supports two person types - DONOR and COMPANY
- **Authentication**: Password hashing, JWT token generation
- **User Profiles**: Basic information (name, email, city, uf, zipcode)
- **Donor Profiles**: Extended with CPF, blood type, birth date
- **Company Profiles**: Extended with CNPJ, institution name, CNES
- **Avatar Management**: Upload and store user avatar images

### Core Entities

**User** ([user.entity.ts](mdc:src/application/core/domain/user.entity.ts))

- `id`: Unique identifier
- `email`: User email address
- `password`: Hashed password
- `name`: User full name
- `city`: User city
- `uf`: State code
- `zipcode`: Postal code (optional)
- `personType`: DONOR or COMPANY
- `avatarPath`: Path to avatar image (optional)

**Donor** ([donor.entity.ts](mdc:src/application/core/domain/donor.entity.ts))

- `id`: Unique identifier
- `cpf`: Brazilian CPF
- `bloodType`: Blood type (A+, A-, B+, B-, AB+, AB-, O+, O-)
- `birthDate`: Date of birth
- `fkUserId`: Foreign key to User

**Company** ([company.entity.ts](mdc:src/application/core/domain/company.entity.ts))

- `id`: Unique identifier
- `cnpj`: Brazilian company registration number
- `institutionName`: Official institution name
- `cnes`: National health establishment code
- `fkUserId`: Foreign key to User

**PersonType** Enum:

- `DONOR`: Individual blood donors
- `COMPANY`: Hospitals, blood centers, health institutions

## Technology Stack

- **Framework**: NestJS 11.x
- **Database**: PostgreSQL with TypeORM
- **Language**: TypeScript 5.7.x
- **Runtime**: Node.js
- **Testing**: Jest
- **Authentication**: JWT (jsonwebtoken)
- **Password Hashing**: bcrypt (via Hash module)
- **File Upload**: Multer (via @nestjs/platform-express)

## Entry Points

- **Main Application**: [main.ts](mdc:src/main.ts)
- **Root Module**: [user.module.ts](mdc:src/user.module.ts)
- **HTTP Port**: Configurable via `process.env.PORT` (default: 3000)

## Environment Configuration

Required environment variables:

- `POSTGRES_HOST`: PostgreSQL host
- `POSTGRES_USERNAME`: Database username
- `POSTGRES_PASSWORD`: Database password
- `POSTGRES_DATABASE`: Database name
- `PORT`: Application port (optional, defaults to 3000)
- `JWT_SECRET`: Secret key for JWT token generation

Configuration loaded via `dotenv/config` in [main.ts](mdc:src/main.ts)

---
> Source: [c3ny/users-service](https://github.com/c3ny/users-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
