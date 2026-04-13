## forfafa

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
Forfafa is a Spring Boot-based OAuth user registration system built with **Kotlin** and Java 21. It supports Naver and Kakao OAuth providers with JWT authentication, uses PostgreSQL as the primary database, and follows Domain-Driven Design (DDD) principles with a clean architecture approach.

### Technology Stack
- **Language**: Kotlin (Spring Boot 3.5.3)
- **Runtime**: Java 21
- **Database**: PostgreSQL (with TestContainers for testing)
- **Security**: Spring Security with JWT
- **OAuth**: WebClient for provider integrations
- **Testing**: JUnit 5, Mockito-Kotlin, TestContainers

## Quick Setup

### Development Scripts
```bash
# Complete local environment setup (Docker + environment)
./scripts/local-setup.sh

# Health check all services
./scripts/health-check.sh
```

### Environment Configuration
```bash
# Copy environment template and configure OAuth credentials
cp .env.example .env
# Edit .env file with your OAuth provider credentials
```

## Common Development Commands

### Build and Run
```bash
# Run application locally
./gradlew bootRun

# Build JAR
./gradlew build

# Run tests
./gradlew test

# Run specific test class
./gradlew test --tests "JwtSecurityIntegrationTest"

# Run integration tests only
./gradlew test --tests "*Integration*"

# Clean build
./gradlew clean build
```

### Docker Commands
```bash
# Start with Docker Compose (local development)
docker-compose up --build

# Start in detached mode
docker-compose up --build -d

# View logs
docker-compose logs -f app

# Stop services
docker-compose down

# Production deployment
docker-compose -f docker-compose.prod.yml up --build -d
```

### Database Commands
```bash
# Connect to PostgreSQL in Docker
docker-compose exec postgres psql -U forfafa_user -d forfafa

# Run database migrations (handled automatically via schema.sql)
# Manual execution if needed:
docker-compose exec postgres psql -U forfafa_user -d forfafa < src/main/resources/schema.sql
```

## Architecture Overview

### Package Structure
The project follows a layered architecture with clear separation of concerns:

- **presentation**: REST controllers and DTOs for API endpoints
  - Controllers handle HTTP requests/responses
  - DTOs define request/response data structures
  - Global exception handling for consistent error responses

- **application**: Application services orchestrating business logic
  - `oauth`: OAuth authentication service handling provider-specific logic
  - `user`: User registration service managing user creation and events
  - Domain events for decoupled communication between services

- **domain**: Core business entities and domain logic
  - `oauth`: OAuth-related value objects and entities (OAuthAccount, OAuthProfile, OAuthProvider)
  - `user`: User aggregate root with value objects (User, Email, UserId)
  - Repository interfaces defining persistence contracts

- **infrastructure**: Technical implementations
  - `oauth`: OAuth client implementations for Naver/Kakao with WebClient
  - `persistence`: JPA entities and repository implementations
  - `security`: JWT token provider and authentication filter

### Key Design Patterns

1. **Domain-Driven Design**: Rich domain models with business logic encapsulated in entities
2. **Repository Pattern**: Abstract persistence layer with interfaces in domain layer
3. **Factory Pattern**: OAuthClientFactory for creating provider-specific clients
4. **Event-Driven**: Domain events (UserRegisteredEvent, OAuthAccountLinkedEvent) for decoupled communication
5. **Value Objects**: Email, UserId for type safety and validation

### Security Architecture

- JWT-based authentication with stateless sessions
- Spring Security filter chain with custom JWT authentication filter
- OAuth2 flow implementation with secure token exchange
- Configurable JWT secret and expiration times

### OAuth Flow

1. Client requests OAuth login URL from `/api/auth/oauth/{provider}`
2. User authenticates with OAuth provider
3. Provider redirects to callback URL with authorization code
4. Application exchanges code for access token
5. Application fetches user profile from provider
6. User is created/updated and JWT is issued

## Environment Configuration

Key environment variables required:
- `NAVER_CLIENT_ID`, `NAVER_CLIENT_SECRET`: Naver OAuth credentials
- `KAKAO_CLIENT_ID`, `KAKAO_CLIENT_SECRET`: Kakao OAuth credentials
- `JWT_SECRET`: Secret key for JWT signing (must be secure in production)
- `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`: Database connection

## Testing Strategy

### Test Structure
The project uses a comprehensive testing approach with Kotlin-specific testing patterns:

- **Unit Tests**: Individual domain objects, value objects, and services
  - Domain entities: `UserTest`, `EmailTest`, `OAuthAccountTest`
  - Application services: `OAuthAuthenticationServiceTest`, `UserRegistrationServiceTest`
  - Infrastructure components: `JwtTokenProviderTest`, `NaverOAuthClientTest`

- **Integration Tests**: End-to-end testing with TestContainers
  - `JwtSecurityIntegrationTest`: JWT authentication flow
  - `OAuthFlowIntegrationTest`: Complete OAuth authentication
  - `OAuthErrorScenariosIntegrationTest`: Error handling scenarios
  - `OAuthPerformanceIntegrationTest`: Performance benchmarks

- **Test Conventions**:
  - Kotlin test classes use descriptive backtick function names
  - MockK for mocking with Kotlin-friendly syntax
  - TestContainers for PostgreSQL integration tests
  - Separate test configurations: `application-test.yml`, `application-integration-test.yml`

### Running Tests
```bash
# Run all tests
./gradlew test

# Run specific test patterns
./gradlew test --tests "*Integration*"
./gradlew test --tests "*Unit*"
./gradlew test --tests "*OAuthFlowIntegrationTest*"

# Performance tests only
./gradlew test --tests "*Performance*"
```

## Kotlin Development Notes

### Code Conventions
- **Domain Layer**: Rich domain models with behavior encapsulated in entities
- **Value Objects**: Use Kotlin data classes for immutable value objects (`Email`, `UserId`)
- **Null Safety**: Leverage Kotlin's null safety features throughout
- **Extension Functions**: Used for domain logic and utility functions
- **Data Classes**: Preferred for DTOs and simple value containers

### Architecture Patterns
- **Repository Pattern**: Interfaces in domain layer, implementations in infrastructure
- **Factory Pattern**: `OAuthClientFactory` for provider-specific OAuth clients
- **Event-Driven**: Domain events (`UserRegisteredEvent`, `OAuthAccountLinkedEvent`)
- **Exception Hierarchy**: Custom exceptions for each layer (domain, application, infrastructure)

### Key Files for Understanding Architecture
- `src/main/kotlin/com/forfafa/forfafa/domain/`: Core business logic
- `src/main/kotlin/com/forfafa/forfafa/application/`: Application services and events
- `src/main/kotlin/com/forfafa/forfafa/infrastructure/`: Technical implementations
- `src/main/kotlin/com/forfafa/forfafa/presentation/`: REST controllers and DTOs

## Important Notes

- Always check OAuth provider configuration status at `/api/auth/oauth/status` before testing OAuth flows
- JWT tokens expire after 24 hours by default (configurable via `JWT_EXPIRATION`)
- Database schema is managed via `schema.sql` - do not use Hibernate auto-DDL in production
- OAuth redirect URIs must exactly match configured values in provider dashboards
- Environment variables must be configured in `.env` file for local development
- Use `./scripts/local-setup.sh` for complete local environment setup
- All tests are written in Kotlin using MockK and follow descriptive naming conventions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/forfafa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/forfafa)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
