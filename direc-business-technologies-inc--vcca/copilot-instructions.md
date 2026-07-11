## vcca

> - .NET 8 · Blazor Server · EF Core (MSSQL) · MediatR · Mapster · Ardalis.Guards · Cookie Authentication · Radzen · B1SLayer (SAP) · Dapper

# VCCA

## Stack
- .NET 8 · Blazor Server · EF Core (MSSQL) · MediatR · Mapster · Ardalis.Guards · Cookie Authentication · Radzen · B1SLayer (SAP) · Dapper

## Solution Map
```
Domain.Entities/                 # DDD aggregates, entities, value objects, domain events, enums
Application.DataTransferObjects/ # DTOs only
Application.UseCases/            # MediatR handlers, IXxxService (internal), Mapster profiles
Shared.Libraries/                # Utilities and extension methods
Shared.DataCipher/               # Data encryption utilities
Database.Libraries/              # EF Core abstractions, ISqlQueryManager
Database.MsSql/                  # DbContext, migrations, repositories
Integration.SAP/                 # SAP HTTP client (outbound only); SQLScripts/ for shared SQL scripts
Web.BlazorServer/                # Blazor Server, Cookie Authentication, Handlers/
```

## Agent Docs
- Building & running the project → @.agent-md/building_the_project.md
- Code conventions → @.agent-md/code_conventions.md
- Service architecture & request pipeline → @.agent-md/service_architecture.md
- Database & EF Core patterns → @.agent-md/database_patterns.md
- Blazor conventions & project structure → @.agent-md/blazor_conventions.md
- Auth (Cookie) → @.agent-md/auth.md
- SAP integration → @.agent-md/sap_integration.md
- Known architectural debts → @.agent-md/architectural_debts.md
- Domain layer internals → @.agent-md/domain_layer.md
- Application layer internals → @.agent-md/application_layer.md
- UI abstractions & workflow → @.agent-md/ui_abstractions.md

---
> Source: [Direc-Business-Technologies-Inc/VCCA](https://github.com/Direc-Business-Technologies-Inc/VCCA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
