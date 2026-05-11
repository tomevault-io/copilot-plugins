## fantasyleague

> **This project operates with auto-approval enabled.**

# FantasyPremierLeague - Claude Instructions

## Permissions

**This project operates with auto-approval enabled.**

Claude Code has permission to:
- ✅ Read any file in the project
- ✅ Write and edit files
- ✅ Create new files and directories
- ✅ Run dotnet CLI commands (build, test, run)
- ✅ Run npm commands for the frontend
- ✅ Execute bash/shell commands within the project directory
- ✅ Start background processes (API, tests)

**Do not ask for permission again** for routine development tasks in this project.

## Command Execution Preferences

**Proceed with reasonable defaults without asking:**
- When running commands that require timeouts, use appropriate values (30-120s)
- When a command fails, retry once with fixes or try alternative approaches
- When multiple options exist, choose the most common/sensible one
- Don't ask "Should I...?" - just do it and report the result

**Only ask the user when:**
- The operation is destructive (delete, drop, force push)
- Multiple significantly different approaches exist with trade-offs
- The request is genuinely ambiguous
- External credentials or configuration is needed

---

## Project Overview

This is a full-stack **Fantasy Premier League** application built with **.NET 9**, **Entity Framework Core**, and **React 18**. It follows **Domain-Driven Design (DDD)** with Clean Architecture principles.

### Technology Stack
- **Backend**: .NET 9, ASP.NET Core Web API, EF Core (In-Memory Database)
- **Frontend**: React 18, Axios, React Router
- **Testing**: xUnit, Moq, AutoFixture, FluentAssertions
- **API Documentation**: Swagger/OpenAPI

---

## Project Structure

```
FantasyPremierLeague/
├── src/
│   ├── Domain/                 # Core business logic (no external dependencies)
│   │   ├── Common/             # Base classes (Entity, IDomainEvent)
│   │   └── Entities/           # Domain entities (Player, Team, GameWeek, MatchResult)
│   │
│   ├── Application/            # Application services & use cases
│   │   ├── DTOs/               # Data Transfer Objects
│   │   ├── Interfaces/         # Repository & service interfaces
│   │   └── Services/           # Application services
│   │
│   ├── Infrastructure/         # Data access & external services
│   │   ├── Persistence/        # EF Core DbContext, seed data
│   │   └── Repositories/       # Repository implementations
│   │
│   ├── WebApi/                 # API controllers & React frontend
│   │   ├── Controllers/        # API controllers (thin, delegate to services)
│   │   ├── ClientApp/          # React application
│   │   └── Program.cs          # Application entry point
│   │
│   ├── McpServer/              # MCP server library (models & tools)
│   ├── McpServer.Api/          # MCP server HTTP API
│   └── McpServer.Host/         # MCP server host process
│
├── tests/
│   ├── UnitTests/              # Unit tests for domain entities and application services
│   │   └── Domain/Entities/    # Domain entity tests (namespace: UnitTests.Domain.Entities)
│   └── IntegrationTests/       # End-to-end tests using real PostgreSQL via Testcontainers
│       └── Fixtures/           # WebApplicationFactory and container setup
│
└── FantasyPremierLeague.sln    # Solution file
```

---

## Architecture Principles

### Dependency Direction
```
WebApi → Application → Domain
              ↑
        Infrastructure
```

### Layer Responsibilities

| Layer | Responsibility |
|-------|----------------|
| **Domain** | Entities, value objects, aggregates, domain events, business rules |
| **Application** | CQRS (Commands, Queries, Handlers), DTOs, repository interfaces |
| **Infrastructure** | Persistence (EF Core), repository implementations, external services |
| **WebApi** | Controllers, request/response contracts, React SPA |

### Key Rules
- **No business logic in controllers** - Controllers must be thin
- **No persistence logic in domain** - Domain is pure C#, no framework dependencies
- **No framework dependencies in domain layer** - Protect domain from infrastructure concerns

---

## Domain Model

### Core Entities

#### Player (Aggregate Root)
- Properties: `Name`, `Position` (GK, DEF, MID, FWD), `Club`, `Price`, `Points`, `GoalsScored`, `Assists`, `CleanSheets`
- Business rules: Price validation (> 0), stats accumulation
- Points calculation: Goals (×5) + Assists (×3) + CleanSheets (×4)

#### Team (Aggregate Root)
- Properties: `Name`, `Manager`, `Budget` (£100m default), `Points`, `Players` collection
- Business rules: Max 15 players, budget constraints
- Contains `TeamPlayer` value objects (junction entity)

#### GameWeek
- Properties: `WeekNumber`, `StartDate`, `EndDate`, `Status` (Upcoming, Active, Completed)
- Business rules: Activation/completion logic

---

## API Endpoints

### Players
- `GET /api/players` - Get all players
- `GET /api/players/{id}` - Get player by ID
- `GET /api/players/position/{position}` - Get players by position
- `POST /api/players` - Create new player
- `PUT /api/players/{id}/stats` - Update player stats
- `DELETE /api/players/{id}` - Delete player

### Teams
- `GET /api/teams` - Get all teams
- `GET /api/teams/{id}` - Get team by ID
- `POST /api/teams` - Create new team
- `POST /api/teams/add-player` - Add player to team
- `DELETE /api/teams/{teamId}/players/{playerId}` - Remove player from team
- `DELETE /api/teams/{id}` - Delete team

---

## Development Workflow

### Running the Application

**Backend:**
```bash
cd src/WebApi
dotnet restore
dotnet run
```
- API runs on: `http://localhost:5000` / `https://localhost:5001`
- Swagger UI: `http://localhost:5000/swagger`

**Frontend:**
```bash
cd src/WebApi/ClientApp
npm install
npm start
```
- React app runs on: `http://localhost:3000`

### Database
- Uses **EF Core In-Memory Database** (data resets on restart)
- Seeded with 15 Premier League players on startup
- To switch to SQL Server: Update `Program.cs` to use `UseSqlServer()`

---

## Testing Guidelines

### Test Framework
- **xUnit** for unit testing
- **Moq** for mocking dependencies
- **AutoFixture** with `AutoMoqCustomization` for data generation (`AutoMockDataAttribute`)
- **FluentAssertions** for assertions
- **Testcontainers** (`Testcontainers.PostgreSql`) for integration tests against a real database

### Naming Convention
All test methods must follow this pattern — no exceptions:
```csharp
{MethodName}_When{Condition}_Should{ExpectedResult}

// Examples:
AddPlayer_WhenTeamHasSpaceAndBudget_ShouldAddPlayer()
GetById_WhenPlayerDoesNotExist_ShouldReturnNotFound()
Constructor_WhenWeekNumberIsZeroOrNegative_ShouldThrowArgumentException()
```

### Which Attribute to Use

| Test type | Attribute | When to use |
|-----------|-----------|-------------|
| Domain entity — single scenario | `[Fact]` | One fixed input, testing a single behaviour (e.g. `Activate_WhenCompleted_ShouldThrow`) |
| Domain entity — data-driven | `[Theory, InlineData(...)]` | Multiple equivalent inputs that vary one parameter (e.g. valid/invalid week numbers) |
| Service / controller with mocks | `[Theory, AutoMockData]` | Dependencies are injected as `Mock<T>` parameters by AutoFixture |
| Integration test | `[Fact]` | Full HTTP stack; no mocking; isolation via `DisposeAsync` |

> **Never** use bare `[Fact]` for service or controller tests that need mocks — use `[Theory, AutoMockData]` so AutoFixture injects the mocks.

### Unit Test Structure (services & controllers)
```csharp
[Theory, AutoMockData]
public async Task MethodName_WhenCondition_ShouldResult(Mock<IRepository> mockRepo)
{
    // Arrange
    mockRepo.Setup(...).ReturnsAsync(...);
    var sut = new ServiceUnderTest(mockRepo.Object);

    // Act
    var result = await sut.MethodAsync(...);

    // Assert
    result.Should()...;
    mockRepo.Verify(..., Times.Once); // only when verifying side-effects
}
```
- Mocks are injected as `Mock<T>` parameters — do **not** use `_fixture.Freeze<>()` or a constructor field
- Always include `// Arrange`, `// Act`, `// Assert` comments
- Use `.Invoking(s => s.MethodAsync(...)).Should().ThrowAsync<TException>()` for exception assertions

### Unit Test Structure (domain entities)
```csharp
// Data-driven: use [Theory, InlineData]
[Theory]
[InlineData(0)]
[InlineData(-1)]
public void Constructor_WhenWeekNumberIsZeroOrNegative_ShouldThrowArgumentException(int weekNumber)
{
    // Arrange
    var start = DateTime.UtcNow;

    // Act
    Action act = () => new GameWeek(weekNumber, start, start.AddDays(7));

    // Assert
    act.Should().Throw<ArgumentException>().WithMessage("*...*");
}

// Single scenario: use [Fact]
[Fact]
public void Activate_WhenCompleted_ShouldThrowInvalidOperationException()
{
    // Arrange / Act / Assert ...
}
```

### Integration Test Structure
Integration tests use `WebApplicationFactory<Program>` backed by a real PostgreSQL container via Testcontainers.

```
tests/IntegrationTests/
├── Fixtures/
│   └── PlayersApiFactory.cs    ← spins up postgres:16-alpine container, runs EnsureCreated
└── PlayersControllerTests.cs   ← [Collection("PlayersApi")], IClassFixture<PlayersApiFactory>, IAsyncLifetime
```

**Rules:**
- Mark the test class with `[Collection("PlayersApi")]` and `IClassFixture<PlayersApiFactory>` so all tests share one container
- Implement `IAsyncLifetime`; wipe the relevant table in `DisposeAsync` (not `InitializeAsync`) for test isolation
- Seed data through a private `SeedXxxAsync` helper that writes directly to `DbContext` — do not seed via HTTP
- Use `[Fact]` (not `[Theory, AutoMockData]`) for integration tests
- Assert both the HTTP status code and the response body shape

```csharp
[Collection("PlayersApi")]
public class PlayersControllerTests : IClassFixture<PlayersApiFactory>, IAsyncLifetime
{
    private readonly HttpClient _client;
    private readonly PlayersApiFactory _factory;

    public PlayersControllerTests(PlayersApiFactory factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }

    public Task InitializeAsync() => Task.CompletedTask;

    public async Task DisposeAsync()
    {
        using var scope = _factory.Services.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<FantasyPremierLeagueDbContext>();
        db.Players.RemoveRange(db.Players);
        await db.SaveChangesAsync();
    }

    private async Task<PlayerDto> SeedPlayerAsync(...) { ... }

    [Fact]
    public async Task GetById_WhenPlayerDoesNotExist_ShouldReturnNotFound()
    {
        var response = await _client.GetAsync($"/api/players/{Guid.NewGuid()}");
        response.StatusCode.Should().Be(HttpStatusCode.NotFound);
    }
}
```

### Test Namespaces
- Service and controller unit tests: `namespace UnitTests;`
- Domain entity unit tests: `namespace UnitTests.Domain.Entities;`
- Integration tests: `namespace IntegrationTests;`

### Running Tests
```bash
dotnet build
dotnet test
```

---

## Coding Patterns

### CQRS Pattern
- **Commands** for write operations (modify state, persist changes)
- **Queries** for read operations (read-only, return DTOs)
- Never mix reads and writes in the same handler

### Repository Pattern
- Interfaces defined in `Application/Interfaces`
- Implementations in `Infrastructure/Repositories`
- EF Core `DbContext` in `Infrastructure/Persistence`

### Domain Events
- Use `IDomainEvent` interface from `Domain/Common`
- Entities inherit from `Entity` base class with event collection
- Dispatch events in application layer after save changes

---

## Common Commands

### .NET CLI
```bash
# Restore packages
dotnet restore

# Build solution
dotnet build

# Run API
dotnet run --project src/WebApi

# Run tests
dotnet test

# Add migration (if using real database)
dotnet ef migrations add InitialCreate --project src/Infrastructure --startup-project src/WebApi
```

### npm (Frontend)
```bash
cd src/WebApi/ClientApp

# Install dependencies
npm install

# Start development server
npm start

# Build for production
npm run build

# Run tests
npm test
```

---

## Code Conventions

### C#
- Use **file-scoped namespaces**
- Use **primary constructors** where appropriate
- Private fields prefixed with `_` (e.g., `_fixture`)
- Async methods suffixed with `Async`
- Use `var` when type is obvious
- Prefer `is null` over `== null`

### React
- Functional components with hooks
- Use `axios` for API calls
- Components in `ClientApp/src/components/`
- Services in `ClientApp/src/services/`

---

## Troubleshooting

### Port Already in Use
- Backend: Edit `src/WebApi/Properties/launchSettings.json`
- Frontend: `PORT=3001 npm start`

### CORS Issues
- Ensure `Program.cs` CORS configuration matches frontend origin
- Default: `http://localhost:3000`

### Database Reset
- Simply restart the application (in-memory DB is cleared)

---

## Future Enhancements

Potential features to add:
- [ ] User authentication (JWT)
- [ ] GameWeek scoring system
- [ ] Player statistics charts
- [ ] Leaderboards
- [ ] Transfer history
- [ ] Team validation rules (position constraints)
- [ ] Persistent database (SQL Server/PostgreSQL)

---

## Important File Locations

| Purpose | Path |
|---------|------|
| Solution file | `FantasyPremierLeague.sln` |
| Backend entry point | `src/WebApi/Program.cs` |
| DbContext | `src/Infrastructure/Persistence/FantasyPremierLeagueDbContext.cs` |
| Domain entities | `src/Domain/Entities/` |
| API Controllers | `src/WebApi/Controllers/` |
| React app | `src/WebApi/ClientApp/` |
| API service (React) | `src/WebApi/ClientApp/src/services/api.js` |
| Unit tests | `tests/UnitTests/` |
| Integration tests | `tests/IntegrationTests/` |

---
> Source: [mahmoufadel/FantasyLeague](https://github.com/mahmoufadel/FantasyLeague) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
