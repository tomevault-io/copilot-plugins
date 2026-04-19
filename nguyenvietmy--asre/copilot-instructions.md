## asre

> - Follow a strict layered DDD structure:


# ============================================================

# Cursor Rules — Domain-Driven Design Architecture

# ============================================================

## GLOBAL ARCHITECTURE

- Follow a strict layered DDD structure:

  - `api/` → controllers, request/response DTOs only.
  - `application/` → commands, queries, handlers, application services.
  - `domain/` → aggregates, entities, value objects, domain services, domain events, repository interfaces.
  - `infra/` → repository implementations, ORM entities, messaging adapters, config, external clients.

- Dependency direction is **one-way**:

  - `api` → `application`
  - `application` → `domain`
  - `infra` → `domain`
  - `domain` → no dependencies.

- Never allow:
  - `domain` importing Spring, JPA, HTTP, logging, RabbitMQ, or infra types.
  - `application` importing infra implementations (only domain interfaces).
  - `api` directly touching domain objects or infra.

---

## DOMAIN LAYER RULES

- Domain classes must contain actual business logic. Avoid anemic domain models.
- No public setters. All state transitions must be through named domain behaviors.
- Aggregates:
  - Expose invariants through methods (`activate()`, `resolveIncident()`, etc.)
  - Never reference other aggregates directly — use IDs.
- Value Objects:
  - Must be immutable and validate on creation.
  - No primitives for domain concepts (use VO instead).
- Domain services:
  - Contain logic that spans multiple aggregates.
  - Must be pure domain logic; no infra calls.
- Domain events:
  - Represent meaningful business occurrences.
  - Stored here as plain objects (no framework annotations).
- Repository interfaces (ports):
  - Declared here.
  - No persistence annotations.

Allowed in `domain/`:

- Aggregates
- Entities
- Value Objects
- Domain Services
- Domain Events
- Repository interfaces

Not allowed:

- Spring components
- JPA/Hibernate annotations except `@Entity` if unavoidable
- DTOs
- Commands/Queries
- HTTP or messaging frameworks
- Mappers

---

## APPLICATION LAYER RULES

- Handles use cases and orchestration only. No business logic.
- Commands & Queries:
  - Simple DTO-like input objects.
  - Belong **exclusively** in `application/` (NOT domain).
- Handlers:
  - `<UseCase>CommandHandler` and `<UseCase>QueryHandler`.
  - Must orchestrate domain logic, never implement domain logic.
  - Inject repository interfaces (not implementations).
- Application services:
  - Coordinate repository calls and domain operations.
  - Convert domain objects → read models, if needed.

Allowed in `application/`:

- Commands
- Queries
- CommandHandlers
- QueryHandlers
- Application services
- Mappers (application ↔ API or application ↔ domain DTOs)

Not allowed:

- Entities
- Repositories implementations
- Controllers
- Framework-specific annotations (except @Service allowed here)
- Business invariants

---

## API LAYER RULES

- Only responsible for translating external requests → internal commands.
- Must not reference domain types directly.
- Must not implement business logic.

Allowed in `api/`:

- Controllers
- Request/Response DTOs
- Exception mappers
- Input validation (e.g., Bean Validation)

Not allowed:

- Commands (must be in application)
- Domain models
- Repository calls
- Business logic

---

## INFRASTRUCTURE LAYER RULES

- Contains actual implementations of domain repository interfaces.
- Handles persistence (JPA/Hibernate), messaging (RabbitMQ/Kafka), Redis, file storage, external APIs.
- Should not implement business rules — only persistence + integration.

Allowed in `infra/`:

- Repository implementations
- ORM Entities
- Spring components (@Repository, @Configuration)
- Mappers (domain ↔ persistence)
- External service adapters

Not allowed:

- Commands/Queries
- Application services
- Domain logic

---

## NAMING CONVENTIONS

- Ports must end with `Port` or `Repository`. Example: `NotificationPublisherPort`.
- Adapter implementations must end with `Adapter` or `RepositoryImpl`.
- Aggregates should end with nothing special; they live in `domain/project/Project.java`.
- Commands: `<UseCase>Command`
- Handlers: `<UseCase>CommandHandler`
- Queries: `<UseCase>Query`
- QueryHandlers: `<UseCase>QueryHandler`
- Domain events: `<EventName>Event`

---

## CQRS GUIDELINES

- All **write** operations use Commands + CommandHandlers.
- All **read** operations use Queries + QueryHandlers.
- Reads must never mutate domain state.
- Writes must always go through aggregates or domain services.

---

## ENFORCEMENT SUMMARY

- Commands stay in `application/`, not domain.
- Domain layer stays pure — no infra leaks.
- API layer never touches domain directly.
- Infra implements ports, never imports application layer.
- No circular imports. No cross-layer pollution.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NguyenVietMy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
