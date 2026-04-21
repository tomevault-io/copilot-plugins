## agent-agnostisk-platform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agent Agnostisk Platform is a .NET 10.0 web application built using Clean Architecture principles with layered separation of concerns. The solution uses .NET Aspire for orchestration and distributed application management.

## Architecture

### Project Structure

The solution follows Clean Architecture with the following layers:

- **AgentAgnostiskPlatform.Domain** - Core business entities and domain logic (innermost layer)
- **AgentAgnostiskPlatform.Application** - Application logic, DTOs, interfaces, validation (FluentValidation)
- **AgentAgnostiskPlatform.Infrastructure** - External concerns and third-party integrations
- **AgentAgnostiskPlatform.Persistence** - Data access layer with Entity Framework Core and SQL Server
- **AgentAgnostiskPlatform.API** - ASP.NET Core Web API presentation layer
- **AgentAgnostiskPlatform.ServiceDefaults** - Shared service configuration for Aspire
- **AgentAgnostiskPlatform.AppHost** - .NET Aspire orchestration host

### Key Architectural Patterns

**Dependency Flow**: API → Infrastructure/Persistence → Application → Domain (dependencies only point inward)

**Entity Base Class**: All domain entities inherit from `BaseEntity` (Backend/src/AgentAgnostiskPlatform.Domain/Entities/BaseEntity.cs) which provides `Id`, `DateCreated`, and `DateModified` properties. The database context automatically sets timestamps on save.

**Generic Repository**: The codebase uses `IGenericRepository<T>` for CRUD operations. Register entity-specific repositories in `Persistence/DependencyInjection.cs` and inject them via the interface from Application layer.

**Request/Response Handlers**: The Application layer defines `IRequestHandler<T>`, `IRequestHandler<TIn, TOut>`, and `IResponseHandler<T>` interfaces for implementing command/query handlers.

**Controller Base**: All API controllers inherit from `AppControllerBase` (Backend/src/AgentAgnostiskPlatform.API/Controllers/Shared/AppControllerBase.cs) which provides consistent API routing (`api/[controller]`), JSON serialization, and lowercase URL transformation.

**Exception Handling**: Global exception handling via:
  - `GlobalExceptionHandler` - Maps domain exceptions to HTTP status codes
  - `ValidationExceptionHandler` - Handles FluentValidation errors
  - `ExceptionMiddleware` - Catches all unhandled exceptions

**DTOs and Validation**: Each DTO has a corresponding FluentValidation validator (e.g., `ManagerCreateRequestDTO` and `ManagerCreateRequestDTOValidator`). Validators are auto-registered via assembly scanning.

## Common Commands

### Build and Run

```bash
# Build the entire solution
dotnet build

# Run the API project directly
dotnet run --project Backend/src/AgentAgnostiskPlatform.API/AgentAgnostiskPlatform.API.csproj

# Run via Aspire AppHost (recommended for development)
dotnet run --project AgentAgnostiskPlatform.AppHost/AgentAgnostiskPlatform.AppHost.csproj
```

### Database Operations

```bash
# Add a new migration (run from Persistence project directory)
cd Backend/src/AgentAgnostiskPlatform.Persistence
dotnet ef migrations add <MigrationName> --startup-project ../AgentAgnostiskPlatform.API/AgentAgnostiskPlatform.API.csproj

# Update database
dotnet ef database update --startup-project ../AgentAgnostiskPlatform.API/AgentAgnostiskPlatform.API.csproj

# Drop database
dotnet ef database drop --startup-project ../AgentAgnostiskPlatform.API/AgentAgnostiskPlatform.API.csproj
```

### Testing

```bash
# Run all tests
dotnet test

# Run tests for a specific project
dotnet test Backend/tests/<TestProject>/<TestProject>.csproj
```

## Development Notes

**Connection String**: Database connection is configured via `AppDatabaseConnectionString` in `appsettings.json`. The application uses SQL Server.

**Culture Settings**: The application uses Danish culture (`da-DK`) globally, affecting number and date formatting.

**CORS**: Currently configured to allow `http://localhost:4200` (likely Angular frontend). Update in `Program.cs` if frontend URL changes.

**Swagger/OpenAPI**: Available in development at `/swagger`. API documentation is generated from XML comments.

**Logging**: Uses Serilog with console and file sinks. Configure in `appsettings.json`.

**Health Checks**: API exposes `/health` endpoint used by Aspire for health monitoring.

## Adding New Features

When adding a new entity:
1. Create entity in Domain layer (inherit from `BaseEntity`)
2. Add DbSet to `AppDatabaseContext`
3. Create entity configuration (implements `IEntityTypeConfiguration<T>`) in Persistence layer
4. Add migration and update database
5. Create DTOs and validators in Application layer
6. Create controller in API layer (inherit from `AppControllerBase`)
7. Register any new repositories in `Persistence/DependencyInjection.cs`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/larsk7cdk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
