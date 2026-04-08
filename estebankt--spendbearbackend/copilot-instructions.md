## spendbearbackend

> SpendBear is a robust personal finance management system architected as a Modular Monolith using Domain-Driven Design (DDD) principles, CQRS, and event-driven architecture.

# SpendBear - Claude Code Context

## Project Overview
SpendBear is a robust personal finance management system architected as a Modular Monolith using Domain-Driven Design (DDD) principles, CQRS, and event-driven architecture.

## Current Context
- **Phase**: Initial Development Setup
- **Priority**: Core module implementation (Identity → Spending → Budgets)
- **Architecture Style**: Modular Monolith with clear module boundaries
- **Event Handling**: Outbox Pattern for reliable event delivery

## Tech Stack
### Backend (.NET 10)
- **Framework**: ASP.NET Core Web API (no minimal APIs)
- **Auth**: Auth0 for authentication
- **ORM**: Entity Framework Core with PostgreSQL (Neon)
- **Caching**: Redis
- **Messaging**: Kafka for event bus
- **Logging**: Serilog
- **API Docs**: Swagger/OpenAPI
- **Testing**: xUnit
- **Monitoring**: Prometheus

### Frontend
- **Framework**: Next.js with TypeScript
- **Deployment**: Vercel
- **Future**: Swift iOS app

### Infrastructure
- **Hosting**: Azure Web App
- **CI/CD**: Azure DevOps (if free tier available)
- **Database**: PostgreSQL on Neon
- **Container Registry**: Azure Container Registry

## Architecture Guidelines

### Module Structure
```
SpendBear/
├── src/
│   ├── Modules/
│   │   ├── Identity/
│   │   │   ├── Identity.Domain/
│   │   │   ├── Identity.Application/
│   │   │   ├── Identity.Infrastructure/
│   │   │   └── Identity.Api/
│   │   ├── Spending/
│   │   │   ├── Spending.Domain/
│   │   │   ├── Spending.Application/
│   │   │   ├── Spending.Infrastructure/
│   │   │   └── Spending.Api/
│   │   ├── Budgets/
│   │   └── Analytics/
│   ├── Shared/
│   │   ├── SpendBear.SharedKernel/
│   │   └── SpendBear.Infrastructure.Core/
│   └── Api/
│       └── SpendBear.Api/
```

### Domain Modeling Principles
1. **Aggregates enforce invariants** - Business rules live in aggregate methods
2. **Value Objects for complex types** - Use Money value object for amounts
3. **Domain Events** - All aggregates emit events via base `AggregateRoot`
4. **Result Pattern** - Use `Result<T>` for command outcomes, avoid exceptions

### CQRS Implementation (No MediatR)
- **Commands**: Vertical slice per feature in `Application/Features/{FeatureName}/`
- **Queries**: Separate read models with projections
- **Handlers**: Direct handler classes, no MediatR pipeline
- Each feature folder contains: Command, Handler, Validator, DTOs

### Event-Driven Architecture
1. **Outbox Pattern**: Store events in `outbox_messages` table before publishing
2. **Event Bus**: Kafka for cross-module communication
3. **Consistency**: 
   - Strong consistency within aggregate boundary
   - Eventual consistency across modules
   - No distributed transactions

### Data Access Patterns
- **Entity Framework Core** configurations in `Infrastructure/Persistence/Configurations/`
- **Repository Pattern** for aggregates only
- **DbContext per module** for isolation
- **Money stored as cents** (long) in database, converted via value object

### Testing Strategy
- **Unit Tests**: Domain logic, invariants, value objects
- **Integration Tests**: Use TestContainers for PostgreSQL
- **E2E Tests**: Full vertical slice testing with in-memory event bus

## Development Guidelines

### Code Style
- **No AutoMapper** - Use explicit mapping methods or simple mappers
- **No MediatR** - Direct handler invocation with dependency injection
- **Explicit over implicit** - Clear method names, avoid magic
- **Feature folders** - Organize by feature, not by type

### Error Handling
- Use `Result<T>` pattern for expected failures
- Domain exceptions only for invariant violations
- Global exception handler for unexpected errors
- Structured error responses with problem details

### Performance Considerations
- Cache hot paths in Redis (dashboards, user profiles)
- Use projections for read-heavy queries
- Batch process CSV imports
- Index strategy: `(user_id, date)` for transactions

### Security
- Auth0 JWT validation
- User context from claims principal
- Row-level security via UserId checks
- No PII in logs

## Current Implementation Status
- [ ] Project structure setup
- [ ] Shared kernel implementation
- [ ] Identity module
- [ ] Spending module (core)
- [ ] Budgets module (reactive)
- [ ] Analytics module (projections)
- [ ] Notifications module
- [ ] Frontend setup

## Known Constraints
- **No distributed transactions** - Use Saga pattern if needed
- **Module communication via events only** - No direct module references
- **Database per module** - Logical separation in same PostgreSQL instance

## Session Notes
_Add notes here during development sessions_

## References
- [Product Requirements](./documentation/PRD.md)
- [Technical Architecture](./documentation/architecture.md)
- [API Design](./documentation/api.md)
- [Task Tracking](./documentation/tasks.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/estebankt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/estebankt)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
