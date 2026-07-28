## extratime

> - All plans are stored in `agents/plans/` directory

# ExtraTime Development Guidelines

## Project Plans

### Plan Location
- All plans are stored in `agents/plans/` directory
- MVP plan: `agents/plans/mvp-plan.md`
- Phase-specific detailed plans: `agents/plans/phase-1-detailed.md`, `agents/plans/phase-2-detailed.md`, etc.
- **When creating a plan in plan mode, always save it to a `.md` file in this directory**

## C# Coding Standards

### Classes
- **Seal all classes by default** - Use `sealed` modifier on all classes unless inheritance is explicitly required
- Base classes and abstract classes should NOT be sealed
- Use primary constructors where applicable
- Prefer records for DTOs, commands, and queries

### Architecture
- **Clean Architecture** with Domain, Application, Infrastructure, API layers
- **Minimal APIs** - No controllers, use static endpoint classes with extension methods
- **Mediator pattern** - Use Mediator source generator (not MediatR)
- **Vertical Slice** organization within Features folder (Commands, Queries, DTOs per feature)

### Naming Conventions
- PascalCase for public members
- _camelCase for private fields
- Async suffix for async methods

### General
- Use file-scoped namespaces
- Prefer `var` when type is obvious
- Use nullable reference types

## Database Conventions

### Naming

| Element | Convention | Example |
|---------|------------|---------|
| Table names | snake_case (plural) | `users`, `league_members` |
| Column names | PascalCase | `Id`, `OwnerId`, `CreatedAt` |
| Primary keys | `PK_tablename` | `PK_leagues` |
| Foreign keys | `FK_table_target_column` | `FK_leagues_users_OwnerId` |
| Indexes | `IX_tablename_column` | `IX_leagues_InviteCode` |

### Data Types
- Enums stored as **strings** with `HasConversion<string>()` and appropriate `MaxLength`
- GUIDs for primary keys (`uuid` in PostgreSQL)
- `timestamp with time zone` for DateTime fields
- JSON stored as `text` for complex types (e.g., array of IDs)

### Configuration
- Use `IEntityTypeConfiguration<T>` classes in `Infrastructure/Data/Configurations/`
- Always specify `ToTable("table_name")` explicitly

## API Conventions

### Endpoints
- Group endpoints in static classes: `{Feature}Endpoints.cs`
- Use `MapGroup()` for route prefixes: `/api/{feature}`
- Private static async methods for handlers
- Use `WithName()` and `WithTags()` for OpenAPI docs

### Validation
- FluentValidation with `AbstractValidator<TCommand>`
- Inject `IValidator<T>` in endpoint methods
- Return `Results.ValidationProblem()` for validation failures

### Error Handling
- Use `Result<T>` pattern from `Application.Common`
- Map domain errors to appropriate HTTP status codes
- Return error objects as `{ error = result.Error }`

## CQRS Conventions

### Commands
- Named `{Action}{Entity}Command` (e.g., `CreateLeagueCommand`)
- Use sealed records
- Implement `IRequest<Result<TDto>>`
- Handler: `{Command}Handler` with primary constructor

### Queries
- Named `Get{Entity}Query` or `Get{Entity}ByIdQuery`
- Use sealed records
- Handler returns `Result<TDto>` or `Result<List<TDto>>`

### DTOs
- Group in `Features/{Feature}/DTOs/{Feature}Dtos.cs`
- Request DTOs: `{Action}{Entity}Request`
- Response DTOs: `{Entity}Dto`, `{Entity}SummaryDto`, `{Entity}DetailDto`

## Code Examples

```csharp
// CORRECT - sealed by default
public sealed class UserService
{
    // ...
}

// CORRECT - record for DTOs
public sealed record LoginRequest(string Email, string Password);

// CORRECT - command with Result pattern
public sealed record CreateLeagueCommand(
    string Name,
    string? Description) : IRequest<Result<LeagueDto>>;

// CORRECT - handler with primary constructor
public sealed class CreateLeagueCommandHandler(
    IApplicationDbContext context,
    ICurrentUserService currentUserService) : IRequestHandler<CreateLeagueCommand, Result<LeagueDto>>
{
    public async ValueTask<Result<LeagueDto>> Handle(CreateLeagueCommand request, CancellationToken ct)
    {
        // ...
    }
}

// CORRECT - EF configuration with snake_case table
public sealed class LeagueConfiguration : IEntityTypeConfiguration<League>
{
    public void Configure(EntityTypeBuilder<League> builder)
    {
        builder.ToTable("leagues");
        builder.Property(l => l.Status)
            .HasConversion<string>()
            .HasMaxLength(20);
    }
}

// CORRECT - base class not sealed (needs inheritance)
public abstract class BaseEntity
{
    public Guid Id { get; set; }
}

// WRONG - missing sealed
public class UserService  // ❌ Should be sealed
{
}
```

---
> Source: [rgonek/ExtraTime](https://github.com/rgonek/ExtraTime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
