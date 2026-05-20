## getting-started-dotnet-aspire

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the companion code repository for the Dometrain course "Getting Started with Aspire". It contains progressive module examples (Modules 03–11), each with multiple sections demonstrating different Aspire concepts. There is no single top-level solution — each module/section has its own `.sln` file.

## Build & Run Commands

Each section is a standalone solution. Navigate to the specific section directory first.

```bash
# Build a specific section
dotnet build "Module 04 - Integrations/Section 04 - Playing with other integrations/Podcasts-Integrations.sln"

# Run the AppHost (orchestrates all services)
dotnet run --project "Module 04 - Integrations/Section 04 - Playing with other integrations/AppHost/AppHost.csproj"

# Run tests (Module 09)
dotnet test "Module 09 - Testing/Section 01 - Testing/Podcasts-Testing.sln"

# Run a single test project
dotnet test "Module 09 - Testing/Section 01 - Testing/AppHost.Tests/AppHost.Tests.csproj"
```

Docker must be running for AppHost projects since Aspire manages containers (SQL Server, Redis, RabbitMQ, MongoDB).

## Tech Stack

- **.NET 9.0** / **Aspire 9.2.0**
- **Entity Framework Core 9.0** with SQL Server
- **xUnit** for testing, **Aspire.Hosting.Testing** for integration tests
- Integrations: SQL Server, Redis, RabbitMQ, MongoDB
- Frontend: Blazor/MVC with Bootstrap 5.3

## Architecture (per section)

Each section follows the same microservices pattern:

| Project | Role |
|---|---|
| **AppHost** | Aspire orchestrator — defines resources, references, and startup ordering |
| **Api** | ASP.NET Core Web API service |
| **Frontend** | ASP.NET Core web app consuming the API via service discovery |
| **Entities** | Shared EF Core DbContext and data models |
| **ServiceDefaults** | Shared OpenTelemetry, resilience, and service discovery configuration |
| **MigrationService** | EF Core database migration worker (added in Module 04+) |

Later modules add: **RatingService** (Module 08 events), **AppHost.Tests** (Module 09), Node.js frontend (Module 11).

## Module Progression

- **Module 03**: Adding Aspire to an existing app (no-Aspire → AppHost → ServiceDefaults)
- **Module 04**: Aspire integrations (DB, seeding, Redis/RabbitMQ/MongoDB)
- **Module 07**: OpenTelemetry (logs, metrics, traces, health checks)
- **Module 08**: Aspire eventing system
- **Module 09**: Testing with Aspire.Hosting.Testing
- **Module 10**: Deployments (Azure Developer CLI, run vs publish, Kubernetes)
- **Module 11**: Extras (custom resource commands, Node.js, Docker, custom URLs)

## Key Patterns

- Services are wired in AppHost via `AddProject<T>()`, connected with `WithReference()`, and ordered with `WaitFor()`
- Container resources use `WithLifetime(ContainerLifetime.Persistent)` to survive restarts
- Secret parameters use `AddParameter("name", secret: true)`
- Frontend exposes external endpoints via `WithExternalHttpEndpoints()`

---
> Source: [Dometrain/getting-started-dotnet-aspire](https://github.com/Dometrain/getting-started-dotnet-aspire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
