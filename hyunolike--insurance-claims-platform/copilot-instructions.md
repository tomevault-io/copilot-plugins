## insurance-claims-platform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Insurance claim automation system - Event-driven backend automating claim processing from submission to disbursement.

- **Stack**: Java 21, Spring Boot 3.x, PostgreSQL 15, Kafka 7.5, Redis 7
- **Architecture**: DDD with hexagonal architecture, event-driven messaging
- **Timeline**: 2025-12-28 ~ 2026-01-12 (16 days)
- **Current Phase**: Phase 1 Complete (Claim Create/Query APIs implemented)

## Build & Run Commands

### Infrastructure
```bash
# Start all services (Kafka, Redis, PostgreSQL)
docker compose up -d

# Stop and clean up
docker compose down -v

# Kafka UI: http://localhost:8081
# PostgreSQL: localhost:5432, DB/User/Password: claim/claim/changeme
```

### Application
```bash
# Build
./gradlew clean build

# Run application
./gradlew bootRun

# Run all tests
./gradlew test

# Run specific test class
./gradlew test --tests com.insurance.claim.domain.model.ClaimTest

# Run specific test method
./gradlew test --tests com.insurance.claim.domain.model.ClaimTest.testClaimCreation
```

## Architecture Principles

### DDD Layered Architecture
The codebase follows strict DDD principles with clear layer separation:

```
api → application → domain ← infrastructure
```

**Critical Rules:**
1. **Domain layer is pure** - No dependencies on Spring, JPA, or infrastructure
2. **Infrastructure adapts to domain** - Domain defines interfaces (e.g., `ClaimRepository`), infrastructure implements them (e.g., `ClaimRepositoryAdapter`)
3. **No circular dependencies** - Dependencies flow inward toward domain
4. **Business logic lives in domain** - `Claim` aggregate contains state transition logic, not services

### Layer Responsibilities

**api/** - REST controllers, DTOs, exception handlers
- `ClaimController`: HTTP endpoints
- `request/`: API request DTOs with validation
- `response/`: API response DTOs
- `ApiExceptionHandler`: Global exception handling

**application/** - Use case orchestration, transaction boundaries
- `ClaimService`: Orchestrates domain operations, manages transactions
- `ClaimMapper`: Maps between API DTOs and domain models
- No business logic here - delegate to domain

**domain/** - Core business logic (framework-agnostic)
- `model/Claim`: Aggregate root with business rules
- `vo/`: Value objects (`ClaimNumber`, `Money`)
- `repository/`: Repository interfaces (implemented by infrastructure)
- `ClaimStatus`: Domain enum (SUBMITTED, IN_REVIEW, APPROVED, REJECTED, PAID)

**infrastructure/** - External system adapters
- `persistence/claim/ClaimEntity`: JPA entity (separate from domain `Claim`)
- `persistence/claim/ClaimJpaRepository`: Spring Data JPA repository
- `persistence/claim/ClaimRepositoryAdapter`: Implements domain `ClaimRepository` interface
- Note: Domain model and JPA entity are separate - adapter maps between them

**config/** - Spring configuration
- `logging/RequestResponseLoggingAspect`: AOP-based API logging

### Key Architectural Patterns

1. **Aggregate Pattern**: `Claim` is an aggregate root managing its own state transitions
2. **Value Objects**: `ClaimNumber` (auto-generated: CLM-YYYYMMDD-XXXXX), `Money` (encapsulates currency)
3. **Repository Pattern**: Domain defines interface, infrastructure implements
4. **Adapter Pattern**: `ClaimRepositoryAdapter` translates between `Claim` and `ClaimEntity`

## Development Workflow

### Adding New Features (Follow This Order)

1. **Start with Domain**
   - Define domain model in `domain/model/`
   - Create repository interface in `domain/repository/`
   - Write domain unit tests (no Spring context needed)

2. **Implement Infrastructure**
   - Create JPA entity in `infrastructure/persistence/`
   - Implement repository adapter
   - Write integration tests

3. **Add Application Service**
   - Create service method in `application/service/`
   - Add DTO mapper in `application/dto/`
   - Write service tests with mocked repositories

4. **Expose via API**
   - Define request/response DTOs in `api/dto/`
   - Implement controller in `api/controller/`
   - Write API integration tests

### Example: Adding a New Entity

```bash
# 1. Domain layer (pure Java)
src/main/java/com/insurance/claim/domain/model/Payment.java
src/main/java/com/insurance/claim/domain/repository/PaymentRepository.java
src/test/java/com/insurance/claim/domain/model/PaymentTest.java

# 2. Infrastructure layer
src/main/java/com/insurance/claim/infrastructure/persistence/payment/PaymentEntity.java
src/main/java/com/insurance/claim/infrastructure/persistence/payment/PaymentJpaRepository.java
src/main/java/com/insurance/claim/infrastructure/persistence/payment/PaymentRepositoryAdapter.java

# 3. Application layer
src/main/java/com/insurance/claim/application/service/PaymentService.java
src/main/java/com/insurance/claim/application/dto/PaymentMapper.java

# 4. API layer
src/main/java/com/insurance/claim/api/controller/PaymentController.java
src/main/java/com/insurance/claim/api/dto/request/PaymentRequest.java
src/main/java/com/insurance/claim/api/dto/response/PaymentResponse.java
```

## Event-Driven Architecture (Upcoming)

The system is being built with event-driven architecture in mind:

**Event Flow**: `claim.requested → claim.reviewed → claim.approved → claim.disbursed`

**Implementation Phases:**
- Phase 2 (Next): Domain events with Spring Events (synchronous)
- Phase 3: Kafka async processing
- Phase 4: Rule Engine + Payment automation

**When implementing events:**
1. Define domain event in `domain/event/` (to be created)
2. Publish from aggregate root
3. Configure Kafka topic in `application.yml`
4. Implement publisher in `infrastructure/messaging/`

## Design Principles

### Idempotency
- API calls must be safe for retries
- Use optimistic locking with `If-Match` headers
- Design for Kafka message reprocessing

### Security & Data Protection
- Encrypt sensitive personal data
- Use business identifiers (e.g., `ClaimNumber`) in API responses, never internal database IDs
- Audit logging for all operations

### Observability
- Every API response includes `traceId`
- Distributed tracing across services
- Log at appropriate levels (DEBUG for `com.insurance.claim`, INFO for root)

## Testing Strategy

### Unit Tests (Domain Layer)
- Pure Java tests, no Spring context
- 100% coverage goal for domain logic
- Example: `ClaimTest`, `MoneyTest`, `ClaimNumberTest`

### Integration Tests (API Layer)
- Full Spring Boot context
- Test controllers with `@SpringBootTest`
- Example: `ClaimServiceTest`

### Test Naming Convention
```java
@Test
void shouldCreateClaimWithValidData() { ... }

@Test
void shouldThrowExceptionWhenAmountIsNegative() { ... }
```

## Commit Conventions

```
<type>: <description>

Types:
- feat: New feature
- fix: Bug fix
- docs: Documentation
- refactor: Code improvement without behavior change
- test: Test-related work
- chore: Build/dependency maintenance
```

## Important Documentation

Consult these files before major changes:
- `docs/IMPLEMENTATION_PLAN.md`: Technical decisions, risk management, phase-by-phase strategy
- `docs/WBS.md`: Detailed work breakdown, Gantt chart, current sprint status
- `docs/PACKAGE_STRUCTURE.md`: Layer-by-layer structure with code examples
- `docs/API.md`: API specifications
- `docs/ERD.md`: Database schema
- `docs/SEQUENCE.md`: Sequence diagrams

## Current Implementation Status

### Completed (Phase 1)
- Domain model: `Claim`, `ClaimStatus`, `Money`, `ClaimNumber`
- JPA infrastructure: `ClaimEntity`, `ClaimJpaRepository`, `ClaimRepositoryAdapter`
- Application service: `ClaimService`
- REST API: `POST /claims`, `GET /claims/{id}`
- AOP logging: `RequestResponseLoggingAspect`
- Unit tests for domain and service layers

### API Endpoints
```
POST /claims          - Create new claim
GET /claims/{id}      - Query claim by ID
GET /health           - Health check
```

### Next Phase (Phase 2)
- Domain events for state transitions
- Event sourcing foundation
- Synchronous event handling with Spring Events

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hyunolike)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hyunolike)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
