## myoniontemplate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

MyOnion is a .NET 10 clean architecture reference API template that demonstrates Domain-Driven Design patterns. The solution showcases a custom lightweight mediator, CQRS pattern, FluentValidation, Mapster for mapping, and EF Core with the Repository/Specification pattern. The project is packaged as a VSIX template for Visual Studio 2022.

## Build and Development Commands

### Local Development
```powershell
# Restore dependencies
dotnet restore MyOnion.sln

# Build the solution (use Release before PRs)
dotnet build MyOnion.sln -c Release

# Run the API with hot reload
dotnet watch run --project MyOnion/src/MyOnion.WebApi/MyOnion.WebApi.csproj

# Apply code formatting
dotnet format MyOnion.sln
```

The API will be available at `https://localhost:5001/swagger` with health checks at `/health`.

### Docker Development
```powershell
# Create development HTTPS certificate
./MyOnion/scripts/Create-DevCert.ps1 -Password "devpassword"

# Start API + SQL Server (requires .env file - copy from .env.example)
docker compose --project-directory MyOnion up --build
```

Access the containerized API at `https://localhost:44378/swagger/index.html`.

### VSIX Template Generation
```powershell
# Build VSIX package (outputs to Desktop by default)
./MyOnion/scripts/Build-OnionTemplate.ps1 -Configuration Release

# Install template locally for testing (copies to Visual Studio templates directory)
./MyOnion/scripts/Install-OnionTemplateLocally.ps1 -Configuration Release
```

The build script:
- Copies all source/test projects to `MyOnion/artifacts/TemplateOnionAPI/`
- Performs token replacement (`MyOnion` → `$safeprojectname$`, namespace updates)
- Normalizes project references (`..\..\src\` → `..\`)
- Generates `MyTemplate.vstemplate` XML for each project
- Creates `TemplateOnionAPI.zip` in `MyOnion/vsix/VSIXTemplateOnionAPI/ProjectTemplates/`
- Builds VSIX using MSBuild if `-SkipVsix` not specified

## Architecture

### Clean Architecture Layers

The solution follows onion architecture with strict dependency flow:

```
WebApi → Infrastructure.Persistence, Infrastructure.Shared, Application → Domain
```

**MyOnion.Domain** (`MyOnion/src/MyOnion.Domain/`)
- Pure domain layer with zero external dependencies
- Contains entities (`Department`, `Employee`, `Position`, `SalaryRange`)
- Value objects (`PersonName`, `DepartmentName`, `PositionTitle`) - sealed, immutable classes with validation
- Base classes: `BaseEntity` (Guid ID), `AuditableBaseEntity` (Created/LastModified tracking)
- Domain events (`EmployeeChangedEvent`, `DepartmentChangedEvent`, etc.)

**MyOnion.Application** (`MyOnion/src/MyOnion.Application/`)
- Business logic layer implementing CQRS pattern
- Custom lightweight mediator (`Messaging/Mediator.cs`)
- Feature-based organization under `Features/<FeatureName>/Commands|Queries/`
- FluentValidation validators co-located with commands/queries
- Mapster mapping profiles in `Mappings/`
- Pipeline behaviors for cross-cutting concerns (validation, caching)
- Repository and specification interfaces

**MyOnion.Infrastructure.Persistence** (`MyOnion/src/MyOnion.Infrastructure.Persistence/`)
- EF Core `ApplicationDbContext` with query tracking disabled by default
- Generic repository implementing `IGenericRepositoryAsync<T>`
- Specialized repositories for domain entities
- Value object ownership configuration (e.g., `OwnsOne()` for `PersonName`)
- Database migrations and seed data

**MyOnion.Infrastructure.Shared** (`MyOnion/src/MyOnion.Infrastructure.Shared/`)
- Cross-cutting services: `IDateTimeService`, `IEmailService`, `IMockDataService`
- Registered with transient lifetime

**MyOnion.WebApi** (`MyOnion/src/MyOnion.WebApi/`)
- ASP.NET Core host with controllers in `Controllers/v1/`
- Service registration extensions in `Extensions/`
- Middleware pipeline: error handling → request timing → Swagger → authentication → CORS → health checks
- Feature flags for authentication (`FeatureManagement:AuthEnabled`) and database mode (`FeatureManagement:UseInMemoryDatabase`)
- JWT authentication configured via `Sts:ServerUrl`, `Sts:Audience`, `Sts:ValidIssuer` (optional when auth disabled)

### Custom Mediator Implementation

**Location:** `MyOnion/src/MyOnion.Application/Messaging/`

The solution uses a custom lightweight mediator instead of MediatR:

```csharp
public interface IRequest<TResponse> { }
public interface IRequestHandler<TRequest, TResponse> where TRequest : IRequest<TResponse>
{
    Task<TResponse> HandleAsync(TRequest request, CancellationToken cancellationToken);
}
public interface IPipelineBehavior<TRequest, TResponse> where TRequest : IRequest<TResponse>
{
    Task<TResponse> HandleAsync(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken);
}
```

**Pipeline Execution:**
- Mediator resolves handler via `IServiceProvider`
- Retrieves all `IPipelineBehavior<,>` registrations
- Builds chain of responsibility in **reverse order**
- Behaviors can intercept, validate, cache, or transform requests

**Registered Behaviors:**
- `ValidationBehavior<TRequest, TResponse>` - Executes FluentValidation validators before handler
- `GetEmployeesCachingDecorator` - Query-specific caching for employee queries
- `GetPositionsCachingBehavior` - Query-specific caching for position queries

### Feature Organization (CQRS)

Features are organized under `MyOnion/src/MyOnion.Application/Features/<FeatureName>/`:

```
Features/
├── Employees/
│   ├── Commands/
│   │   ├── CreateEmployee/
│   │   │   ├── CreateEmployeeCommand.cs          # Command with nested Handler class
│   │   │   └── CreateEmployeeCommandValidator.cs # FluentValidation rules
│   │   ├── UpdateEmployee/
│   │   └── DeleteEmployeeById/
│   └── Queries/
│       └── GetEmployees/
│           ├── GetEmployeesQuery.cs               # Query with nested Handler class
│           ├── GetEmployeesQueryValidator.cs
│           ├── GetEmployeesViewModel.cs           # DTO for response
│           └── GetEmployeesCachingDecorator.cs    # Optional caching behavior
```

**Pattern:**
- Each command/query implements `IRequest<Result>` or `IRequest<Result<T>>`
- Handler is nested class implementing `IRequestHandler<TRequest, TResponse>`
- Validator extends `AbstractValidator<TRequest>`
- Returns `Result`, `Result<T>`, or `PagedResult<T>` wrapper

**Example handler invocation:**
```csharp
var result = await _mediator.Send(new CreateEmployeeCommand { ... }, cancellationToken);
```

### Repository and Specification Pattern

**Generic Repository Interface:** `MyOnion/src/MyOnion.Application/Interfaces/IGenericRepositoryAsync.cs`

Key methods:
- `GetByIdAsync(Guid id)` - Single entity retrieval
- `GetPagedReponseAsync(int pageNumber, int pageSize)` - Basic pagination
- `GetAllShapeAsync(string fields, string orderBy)` - Dynamic field selection
- `ListAsync(ISpecification<T>)` - Query by specification
- `FirstOrDefaultAsync(ISpecification<T>)` - Single entity by specification
- `CountAsync(ISpecification<T>)` - Count entities matching specification
- `BulkInsertAsync(IEnumerable<T>)` - Batch operations

**Specification Pattern:** Uses Ardalis.Specification library

MyOnion uses the battle-tested [Ardalis.Specification](https://github.com/ardalis/Specification) library (v9.3.1) for implementing the Specification pattern. Specifications are located in `MyOnion/src/MyOnion.Application/Specifications/<EntityName>/`.

Specifications encapsulate query logic using a fluent builder API:
```csharp
public class SalaryRangesByFiltersSpecification : Specification<SalaryRange>
{
    public SalaryRangesByFiltersSpecification(GetSalaryRangesQuery request, bool applyPaging = true)
    {
        // Filtering - type-safe with compile-time validation
        if (!string.IsNullOrWhiteSpace(request.Name))
        {
            var term = request.Name.Trim();
            Query.Where(s => s.Name.Contains(term));
        }

        // Ordering - expression-based
        Query.OrderBy(s => s.Name);

        // Pagination
        if (applyPaging && request.PageSize > 0)
        {
            Query.Skip((request.PageNumber - 1) * request.PageSize)
                 .Take(request.PageSize);
        }

        // EF Core optimizations
        Query.AsNoTracking()
             .TagWith("GetSalaryRangesByFilters");
    }
}
```

Specifications support:
- **Type-safe filtering:** `Query.Where(e => e.Name.Contains("Smith"))`
- **Expression-based ordering:** `Query.OrderBy(e => e.LastName)`
- **Eager loading:** `Query.Include(e => e.Position).ThenInclude(p => p.Department)`
- **Pagination:** `Query.Skip(10).Take(20)`
- **EF Core features:** `AsNoTracking()`, `AsSplitQuery()`, `TagWith()`, `IgnoreQueryFilters()`
- **Single result marker:** Inherit from `SingleResultSpecification<T>` for queries returning one entity
- **Search helper:** `Query.Search(e => e.Email, $"%{term}%")`

The repository uses `Ardalis.Specification.EntityFrameworkCore`'s `SpecificationEvaluator` to translate specifications into EF Core queries.

### Mapster Configuration

**Location:** `MyOnion/src/MyOnion.Application/Mappings/GeneralProfile.cs`

Mapster configuration is registered as a singleton using assembly scanning:

```csharp
var mapsterConfig = TypeAdapterConfig.GlobalSettings;
mapsterConfig.Scan(Assembly.GetExecutingAssembly());
services.AddScoped<IMapper, ServiceMapper>();
```

**Mapping Patterns:**

BiDirectional mapping:
```csharp
config.NewConfig<SalaryRange, GetSalaryRangesViewModel>().TwoWays();
```

Value object property extraction:
```csharp
config.NewConfig<Employee, GetEmployeesViewModel>()
    .Map(dest => dest.FirstName, src => src.Name.FirstName)
    .Map(dest => dest.LastName, src => src.Name.LastName);
```

Value object construction:
```csharp
config.NewConfig<CreateEmployeeCommand, Employee>()
    .Map(dest => dest.Name, src => new PersonName(src.FirstName, src.MiddleName, src.LastName));
```

### Result Pattern

**Location:** `MyOnion/src/MyOnion.Application/Common/Results/Result.cs`

All handlers return `Result` or `Result<T>` wrappers:

```csharp
public class Result
{
    public bool IsSuccess { get; }
    public string Message { get; }
    public IReadOnlyCollection<string> Errors { get; }
    public double? ExecutionTimeMs { get; }
}

public class Result<T> : Result
{
    public T Value { get; }
}

public class PagedResult<T> : Result<T>
{
    public int PageNumber { get; }
    public int PageSize { get; }
    public int RecordsFiltered { get; }
    public int RecordsTotal { get; }
}
```

### Caching Strategy

**Pipeline Behaviors:**
- `GetEmployeesCachingDecorator` - Caches employee query results with TTL expiration
- `GetPositionsCachingBehavior` - Caches position query results

**Cache Key Pattern:**
Cache keys are built from normalized query parameters:
```csharp
var key = $"{CacheKeyPrefixes.EmployeesAll}:{pageNumber}:{pageSize}:{filters.ToLower().Trim()}:{fields.ToLower()}";
```

**Cache Invalidation:**
`CacheInvalidationEventHandler` subscribes to domain events and invalidates cache prefixes:
- `EmployeeChangedEvent` → Invalidate `EmployeesAll` and `DashboardMetrics`
- `PositionChangedEvent` → Invalidate `PositionsAll` and `DashboardMetrics`
- Etc.

## Coding Guidelines

### Feature Development

When adding a new feature:

1. **Create feature folder:** `MyOnion/src/MyOnion.Application/Features/<FeatureName>/`
2. **Add commands/queries:**
   - Create `Commands/<CommandName>/<CommandName>Command.cs` with nested `Handler` class
   - Create `<CommandName>CommandValidator.cs` extending `AbstractValidator<T>`
   - Return `Result<T>` from handler
3. **Add DTOs:** Create ViewModels/DTOs in `Queries/<QueryName>/` or `DTOs/`
4. **Configure mapping:** Add Mapster configuration in `GeneralProfile.cs`
5. **Add controller endpoint:** Create controller in `MyOnion.WebApi/Controllers/v1/` using `[ApiVersion("1.0")]`

Example controller pattern:
```csharp
[ApiVersion("1.0")]
public class EmployeesController : BaseApiController
{
    [HttpGet]
    public async Task<IActionResult> Get([FromQuery] GetEmployeesQuery query)
    {
        var result = await Mediator.Send(query);
        return result.IsSuccess ? Ok(result) : BadRequest(result);
    }
}
```

### Value Objects

When creating value objects:
- Use sealed, immutable classes
- Add validation in constructor
- Implement `Equals()` and `GetHashCode()` for value equality
- Configure as owned entities in EF Core using `OwnsOne()`

Example from `PersonName.cs:23-35`:
```csharp
public sealed class PersonName
{
    public string FirstName { get; }
    public string LastName { get; }
    public string FullName => $"{FirstName} {LastName}";

    private PersonName() { } // EF Core
    public PersonName(string firstName, string lastName)
    {
        FirstName = Normalize(firstName);
        LastName = Normalize(lastName);
    }
}
```

### Database Configuration

EF Core configuration is in `ApplicationDbContext.cs` and `ApplicationDbContextHelpers.cs`:

- Use `OwnsOne()` for value objects to store as columns in parent table
- Configure cascade delete behavior explicitly (default is Restrict for critical entities)
- Set decimal precision: `HasColumnType("decimal(18, 2)")`
- Add indexes on foreign keys and frequently queried columns
- Override `SaveChangesAsync()` to automatically set audit fields

### Validation

FluentValidation rules are co-located with commands/queries:

```csharp
public class CreateEmployeeCommandValidator : AbstractValidator<CreateEmployeeCommand>
{
    public CreateEmployeeCommandValidator()
    {
        RuleFor(e => e.FirstName)
            .NotEmpty().WithMessage("{PropertyName} is required.")
            .MaximumLength(100).WithMessage("{PropertyName} must not exceed 100 characters.");

        RuleFor(e => e.Email)
            .EmailAddress().WithMessage("{PropertyName} must be a valid email.");
    }
}
```

The `ValidationBehavior` pipeline behavior automatically executes validators before handlers.

### Security and Configuration

- **Never commit secrets:** Use `dotnet user-secrets` locally or environment variables in Docker/CI
- **JWT Configuration:** Set `Sts:ServerUrl`, `Sts:Audience`, `Sts:ValidIssuer` in `appsettings.Development.json` or user secrets (only required when `FeatureManagement:AuthEnabled` is true)
- **CORS:** Configure allowed origins in `Cors:AllowedOrigins` (JSON array) - the default open policy is only for local HTTPS development
- **Database:** Connection string is in `appsettings.json` - override with environment variable `ConnectionStrings__DefaultConnection`

### Commit Conventions

Follow imperative commit message style (examples from git log):
- "Update build script"
- "Add EasyCaching blog"
- "Minor update to blog"

Keep subject under 72 characters, start with a verb, and add scope if needed. PRs should include summary, validation steps, and screenshots/sample responses for API contract changes.

### Documentation Naming Convention

All documentation files in `MyOnion/docs/` follow a consistent kebab-case naming convention with category prefixes:

**Format:** `{category}-{descriptive-name}.md`

**Categories:**
- `plan-` - Planning documents for features, migrations, and upgrades
- `blog-` - Blog posts and articles (published or draft)
- `design-` - Technical design and architecture documentation
- `release-notes-` - Release notes for versioned releases
- `github-release-` - GitHub-specific release content
- `test-` - Testing plans and test strategy documents

**Examples:**
- `plan-net10-upgrade.md` - .NET 10 upgrade planning document
- `blog-template-onion-api-net10.md` - Blog post about .NET 10 template
- `design-value-object.md` - Value object design documentation
- `release-notes-10.1.3.md` - Release notes for v10.1.3
- `test-coverage-100-percent.md` - Test coverage planning

**Benefits:**
- Alphabetically groups by category when sorted
- Consistent lowercase for git-friendliness
- Easy to search and filter by category
- URL-friendly for potential wiki/docs sites

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/workcontrolgit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
