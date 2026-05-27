## quarkus-kotlin-clean-arch-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

```bash
# Development mode with hot reload
./gradlew :web-api:quarkusDev

# Build all modules
./gradlew clean build

# Run tests
./gradlew test

# Build uber-jar for production
./gradlew clean :web-api:build -Dquarkus.package.type=uber-jar -Dquarkus.profile=release

# Build native image
./gradlew clean :web-api:build -Dquarkus.package.type=native -Dquarkus.profile=release

# Docker build (after building uber-jar)
docker build -f web-api/src/main/docker/Dockerfile.uber-jar -t quarkus/web-api .
docker run -i --rm -p 8080:8080 quarkus/web-api
```

## Architecture Overview

This is a **Quarkus Kotlin** project implementing **Clean Architecture** with **Hexagonal (Ports & Adapters)** pattern.

### Module Structure

```
quarkus-kotlin-clean-arch-template/
├── core/domain           # Pure domain entities (no dependencies)
├── core/application         # Business logic, defines ports (interfaces)
├── common               # Adapter implementations, infrastructure
└── web-api              # REST API entry point, inbound adapters
```

### Dependency Flow

```
HTTP Request
  → UserResource (web-api/adapter/in/gateway)
    → CreateUserUseCase (core/application/port/in)
      → UserRepository (core/application/port/out) [INTERFACE]
        ← UserRepositoryImpl (common/adapter/out) [IMPLEMENTATION]
          → UserJpaRepository (Spring Data JPA)
            → PostgreSQL
```

**Key Principle:** Core layer defines interfaces (ports), outer layers implement them (adapters). Dependencies point inward.

### Layer Responsibilities

1. **core/domain**: Pure business entities, no framework dependencies
2. **core/application**:
   - Business logic orchestration
   - Defines **Use Case** (use cases)
   - Defines **Port In** (themselves)
   - Defines **Port Out** (Repository, Service interfaces)
3. **common**:
   - Implements Port Out interfaces
   - Contains adapters for external systems (DB, REST clients)
   - Works with DAOs (JPA entities)
   - Provides mappers (Domain Entity ↔ DAO)
4. **web-api**:
   - HTTP/REST endpoints
   - Converts REST DTOs ↔ Use Case inputs/outputs
   - Exception handlers
   - Security/authorization

## Key Architectural Patterns

### 1. Ports and Adapters (Dependency Inversion)

**Ports** (interfaces) defined in `core/application/port/out/`:
```kotlin
interface UserRepository : CommonRepository<User, UserId>
interface AuthService { fun authenticate(...) }
```

**Adapters** (implementations) in `common/adapter/out/`:
```kotlin
@ApplicationScoped
class UserRepositoryImpl : UserRepository { ... }

@ApplicationScoped
class AuthServiceImpl : AuthService { ... }
```

### 2. Three-Domain Object Pattern

- **DAO** (`UserDAO`): JPA entity with `@Entity`, database schema
- **DTO** (`UserResourceRequest/Response`): REST API contract

**Mappers** convert between layers (e.g., `UserMapper.toDomainEntity()`)

### 3. Use Case Pattern

Each use case is an independent class with:
- Input DTO (e.g., `CreateUserUseCaseInput`)
- Output DTO (e.g., `GetUserInfoUseCaseOutput`)
- Injected dependencies (ports only)
- `execute()` method

Example:
```kotlin
class CreateUserUseCase(
    private val userRepository: UserRepository  // Port interface
) {
    fun execute(input: CreateUserUseCaseInput) {
        // Business logic here
        userRepository.save(domainEntity = user)
    }
}
```

### 4. Dependency Injection via CDI

Use cases are wired in `common/config/UseCaseConfig.kt`:
```kotlin
@Produces
fun createUserUseCase(userRepository: UserRepository): CreateUserUseCase {
    return CreateUserUseCase(userRepository = userRepository)
}
```

Quarkus CDI automatically injects `UserRepositoryImpl` where `UserRepository` is needed.

### 5. Standardized Response Handling

All REST endpoints use `ResponseHandler`:
```kotlin
// Success
ResponseHandler.ok(data = userData)
// Returns: { "data": {...} }

// Error
ResponseHandler.unprocessableEntity(code = "USER_NOT_FOUND", message = "...")
// Returns: { "error": { "code": "...", "message": "..." } }
```

### 6. Exception Mapping

Domain exceptions thrown from use cases are caught by `@Provider` exception handlers:
```kotlin
@Provider
class UserNotFoundExceptionHandler : ExceptionMapper<UserNotFoundException> {
    override fun toResponse(exception: UserNotFoundException): Response {
        return ResponseHandler.unprocessableEntity(...)
    }
}
```

## Important Conventions

### When Adding New Features

1. **Define port interface** in `core/application/port/out/` if external dependency needed
2. **Create use case** in `core/application/use_case/`
3. **Implement adapter** in `common/adapter/out/`
4. **Wire use case** in `common/config/UseCaseConfig.kt` using `@Produces`
5. **Add REST resource** in `web-api/adapter/in/gateway/`
6. **Create exception handler** in `web-api/handler/` if new exceptions

### Security

- JWT authentication via Keycloak configured in `application.yml`
- Authorization via `@PermissionsAllowed(ResourcePermissionChecker.SCOPE_USER)`
- JWT tokens read from cookie (configurable via `APPLICATION.AUTH.COOKIE_NAME`)

### Database

- PostgreSQL with Liquibase migrations in `web-api/src/main/resources/db/`
- Devservices auto-provision database in dev mode
- Sensitive fields (e.g., email) encrypted via `DAOAttributeConverter` using AES
- Spring Data JPA repositories in `common/adapter/out/jpa_repository/`

### Configuration Profiles

- `local`: Development mode with devservices (port 5433)
- `test`: Test mode with devservices (port 5434)
- `release`: Production configuration

Environment variables referenced via `${VAR_NAME}` syntax in `application.yml`.

## Technology Stack

- **Framework**: Quarkus 3.27.1
- **Language**: Kotlin 2.2.0, Java 21
- **Build**: Gradle 8.13 with Kotlin DSL
- **Database**: PostgreSQL with Liquibase
- **ORM**: Hibernate ORM with Spring Data JPA
- **REST**: Quarkus REST with Jackson
- **Security**: SmallRye JWT, Keycloak OIDC
- **API Docs**: SmallRye OpenAPI (Swagger)
- **Testing**: JUnit5, REST Assured
- **Monitoring**: Sentry logging

## Module Dependencies

```
web-api
  ├── depends on: core:application, common

common
  ├── depends on: core:application

core:application
  ├── depends on: core:domain

core:domain
  └── no dependencies (pure Kotlin)
```

**Critical**: Never create dependencies that violate this flow (e.g., core cannot depend on common or web-api).

---
> Source: [axuanhogan/quarkus-kotlin-clean-arch-template](https://github.com/axuanhogan/quarkus-kotlin-clean-arch-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
