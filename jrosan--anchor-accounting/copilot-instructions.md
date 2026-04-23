## anchor-accounting

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Enterprise accounting software: .NET 10 backend (C#) with React Native/Expo mobile app (TypeScript). Implements Domain-Driven Design, CQRS via MediatR, Clean Architecture, and Repository/Unit of Work patterns.

## Build & Run Commands

### Backend (.NET)
```bash
dotnet restore
dotnet build
dotnet test

# Run API server
cd src/AnchorAccounting.API && dotnet run

# EF Core migrations (always specify both projects)
dotnet ef migrations add <Name> --project src/AnchorAccounting.Infrastructure --startup-project src/AnchorAccounting.API
dotnet ef database update --project src/AnchorAccounting.Infrastructure --startup-project src/AnchorAccounting.API
```

### Mobile (React Native/Expo)
```bash
cd mobile
npm install
npm start          # Expo dev server
npm run android    # Android
npm run ios        # iOS
npm test           # Jest
npm run lint       # ESLint
```

## Architecture

Four-layer Clean Architecture with strict dependency flow (inner layers have no knowledge of outer layers):

1. **Domain** (`src/AnchorAccounting.Domain/`) - Entities with private setters and factory methods, Value Objects (Money, TaxId), domain events, repository interfaces, specifications. No external dependencies.
2. **Application** (`src/AnchorAccounting.Application/`) - CQRS commands/queries with MediatR handlers, FluentValidation validators (auto-wired via `ValidationBehavior` pipeline), DTOs, `Result<T>` pattern for operation outcomes.
3. **Infrastructure** (`src/AnchorAccounting.Infrastructure/`) - EF Core `AccountingDbContext` with SQL Server, repository implementations, domain event dispatching in `SaveChangesAsync`, Identity/JWT configuration.
4. **API** (`src/AnchorAccounting.API/`) - REST controllers inheriting `BaseController`, `[Authorize]` with role-based access (Admin, Accountant, Cashier, Manager), global exception middleware.

DI is configured via `DependencyInjection.cs` in both Application and Infrastructure layers, composed in `Program.cs`.

## Key Patterns to Follow

- **Entities**: Use private setters, `Create()` factory methods, emit domain events on state changes. EF Core requires a private parameterless constructor.
- **Commands/Queries**: Return `Result<T>` (not exceptions) for business failures. Validators are auto-discovered by FluentValidation and run via MediatR pipeline.
- **Repositories**: Define interface in Domain, implement in Infrastructure. Always go through `IUnitOfWork` for persistence. All repositories must be exposed via `IUnitOfWork`.
- **Controllers**: Must inherit `BaseController` (not `ControllerBase`). Inject `IMediator` and use `_mediator.Send()` for all operations. Use `[Authorize]` at class level; add `[Authorize(Roles = "...")]` on sensitive endpoints. Call `GetCurrentUserId()` from `BaseController`.
- **Domain Event Handlers**: Implement `INotificationHandler<TEvent>`. Wrap logic in try/catch with logging. For financial state changes (post/pay/void), create corresponding journal entries. For all operations, create audit log entries via `IAuditService`. Email notifications via `IEmailService` (currently logs, ready for SMTP integration).
- **Validators**: Create in `Application/Validators/`. Use FluentValidation `AbstractValidator<T>`. Auto-discovered by MediatR pipeline — no manual registration needed.

## Testing

Two test projects in `tests/`:
- `AnchorAccounting.Domain.Tests` — Entity and value object tests (xUnit + FluentAssertions)
- `AnchorAccounting.Application.Tests` — Handler and validator tests (xUnit + Moq + FluentAssertions)

Run all: `dotnet test`. Run single project: `dotnet test tests/AnchorAccounting.Domain.Tests`.

## Pipeline Behaviors

MediatR pipeline processes requests in this order:
1. `LoggingBehavior` — logs request name before/after
2. `ValidationBehavior` — runs FluentValidation validators
3. `PerformanceBehavior` — warns if execution > 500ms
4. `TransactionBehavior` — wraps Commands (not Queries) in DB transactions

## Infrastructure Endpoints

- `/health` — EF Core database health check
- Rate limiting: 100 req/min global, 10 req/min on `/api/auth/*`
- `IEmailService` — injected in event handlers, currently logs (swap `EmailService` for SMTP implementation in production)
- Environment configs: `appsettings.Development.json` (Mattie SQL Server), `appsettings.Staging.json`, `appsettings.Production.json`

## Database

SQL Server LocalDB for development. Database seeds default users on startup via `DatabaseSeeder` (admin@example.com, accountant@example.com). Connection string in `appsettings.json`.

## Mobile App

Field-focused subset (not desktop replica): dashboards, expense capture, invoice/bill management, customer/vendor browsing, financial reports, payment collection. Uses Expo Router file-based routing (`app/(tabs)/`), Zustand for client state, TanStack Query for server state, Axios for API calls, Expo SecureStore for JWT tokens. Role-based feature gating via `usePermissions` hook. API URL configurable via `app.json` extra config. Error boundary wraps the app root.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JRosan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
