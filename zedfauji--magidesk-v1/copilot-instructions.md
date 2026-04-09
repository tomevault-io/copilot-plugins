## magidesk-v1

> - **Domain Layer**: Zero dependencies on other layers. No EF Core, no HTTP, no file I/O.

# Architecture Rules

## Clean Architecture Enforcement

### Layer Dependencies
- **Domain Layer**: Zero dependencies on other layers. No EF Core, no HTTP, no file I/O.
- **Application Layer**: Depends ONLY on Domain. No infrastructure dependencies.
- **Infrastructure Layer**: Depends on Application and Domain. Implements interfaces defined in Application/Domain.
- **Presentation Layer**: Depends ONLY on Application. Never directly accesses Domain or Infrastructure.

### Dependency Direction
```
Presentation → Application → Domain
Infrastructure → Application → Domain
```

**VIOLATION**: If any layer depends on a layer further out, this is an architectural violation.

### Project References
- Domain: No project references
- Application: References Domain only
- Infrastructure: References Application and Domain
- Presentation: References Application only
- Tests: Reference their respective projects

## Domain Layer Rules

### Pure Business Logic
- Domain entities contain business logic, not just data
- All business rules enforced in Domain layer
- No external dependencies (no EF Core, no HTTP, no file I/O)
- Language-agnostic business rules

### Entity Design
- Use rich domain model (entities have behavior)
- Aggregate roots: Ticket, CashSession, Shift
- Value objects: Money, UserId (immutable)
- Domain events for important state changes
- Invariants enforced at construction and mutation

### Domain Services
- Complex business logic that doesn't belong in a single entity
- Stateless services
- Examples: TicketDomainService, PaymentDomainService, DiscountDomainService

## Application Layer Rules

### Use Cases
- One use case per file
- Commands for write operations
- Queries for read operations
- Use cases orchestrate domain logic
- Transaction boundaries defined here

### DTOs
- All communication with Presentation layer uses DTOs
- Never expose domain entities to UI
- DTOs are data containers only (no behavior)
- Use mapping (AutoMapper or manual) between entities and DTOs

### Interfaces
- Repository interfaces defined here
- Application service interfaces defined here
- Infrastructure implements these interfaces

## Infrastructure Layer Rules

### Repository Pattern
- Implement repository interfaces from Application layer
- Use EF Core for data access
- No business logic in repositories
- Return domain entities, not DTOs

### EF Core Configuration
- Entity configurations in separate files
- Value object mappings configured here
- Optimistic concurrency with Version fields
- Database constraints as last line of defense

### External Services
- Payment gateway implementations
- Printer service implementations
- All external integrations here

## Presentation Layer Rules

### MVVM Pattern
- Views: XAML only, code-behind minimal (event handlers only)
- ViewModels: Thin coordinators, delegate to Application layer
- Models: DTOs from Application layer

### Zero Business Logic
- ViewModels contain NO business logic
- ViewModels call Application layer use cases
- ViewModels handle UI state and user input only
- All validation through Application layer

### No Direct Database Access
- ViewModels never use DbContext
- ViewModels never use repositories directly
- All data through Application layer interfaces

## Database Rules

### PostgreSQL
- Database: `magidesk_pos`
- Schema: `magidesk` (use this schema for all tables)
- Connection: Local passwordless PostgreSQL
- EF Core migrations for all schema changes

### Naming Conventions
- Tables: PascalCase (e.g., `Tickets`, `OrderLines`)
- Columns: PascalCase (e.g., `Id`, `TicketNumber`, `CreatedAt`)
- Foreign Keys: `{Entity}Id` (e.g., `TicketId`, `UserId`)
- Indexes: `IX_{Table}_{Columns}`
- Primary Keys: `PK_{Table}`

### Migrations
- All schema changes via EF Core migrations
- Never modify database directly
- Migrations must be reversible
- Test migrations on development database first

## State Management

### Ticket State Machine
- Use enum for TicketStatus (not multiple booleans)
- Valid transitions only:
  - Draft → Open
  - Open → Paid
  - Paid → Closed
  - Any → Voided (if no payments)
  - Closed → Refunded
- State transitions enforced in domain

### Payment State
- Use enum for PaymentStatus
- Card payments: Pending → Authorized → Captured
- Cash payments: Pending → Completed
- All payments can be Voided or Refunded

## Error Handling

### Domain Exceptions
- BusinessRuleViolationException: Invariant violation
- InvalidOperationException: Operation not allowed in current state
- ConcurrencyException: Optimistic concurrency conflict

### Application Exceptions
- ValidationException: Use case validation failed
- NotFoundException: Entity not found
- UnauthorizedException: Permission denied

### Presentation Exceptions
- Display user-friendly error messages
- Log technical details
- Never expose stack traces to users

## Testing Requirements

### Domain Layer
- >90% test coverage required
- All invariants must have unit tests
- Domain services must be tested
- Value objects must be tested

### Application Layer
- >80% test coverage required
- All use cases must be tested
- Mock repositories in tests
- Test validation logic

### Integration Tests
- Test repository implementations
- Test EF Core configurations
- Test database constraints

## Code Organization

### Folder Structure
```
Domain/
  Entities/
  ValueObjects/
  DomainServices/
  DomainEvents/
  Enumerations/
  Exceptions/

Application/
  Commands/
  Queries/
  DTOs/
  Interfaces/
  Services/

Infrastructure/
  Persistence/
    Configurations/
    Repositories/
    DbContext/
  Services/
  External/

Presentation/
  Views/
  ViewModels/
  Converters/
  Resources/
```

## Prohibited Patterns

### NEVER:
- Put business logic in ViewModels
- Access database from Presentation layer
- Use domain entities in ViewModels
- Skip invariant enforcement
- Modify closed/voided/refunded tickets
- Hard delete financial records
- Use string-based status (use enums)
- Use multiple boolean flags for state (use state machine)
- Copy code from FloreantPOS (reference only)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zedfauji)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zedfauji)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
