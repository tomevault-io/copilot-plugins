## enterprise-app

> A Docker-first, event-driven enterprise demo application focused around a generic "model," deployed to Azure via Terraform (and respective CLIs). The system demonstrates enterprise patterns: SSO, async job processing, observability, and repeatable IaC deployments.

# CLAUDE.md — Enterprise App

## Project Overview

A Docker-first, event-driven enterprise demo application focused around a generic "model," deployed to Azure via Terraform (and respective CLIs). The system demonstrates enterprise patterns: SSO, async job processing, observability, and repeatable IaC deployments.

The development occurs within a VS Code-based Devcontainer, as defined in `.devcontainer`. Any missing CLIs, dependencies, should be updated accordingly in setup script `.devcontainer/scripts/setup-env.sh`.

### Architecture

- **Angular 20 SPA** (`ui/`) — client UI, hosted on Azure Static Web Apps
- **ASP.NET Core .NET 10 REST API** (`api/`) — system-of-record, EF Core + Postgres, publishes commands to RabbitMQ
- **Python 3 Data Engine** (`data-engine`) - companion service for numerical computations and data-related workflows, jobs, etc.; transmits data via RabbitMQ
- **RabbitMQ** — message broker for async job workflows
- **PostgreSQL** — relational data store, managed via EF Core migrations
- **Azure Container Apps** — runtime for API and RabbitMQ containers
- **Terraform** (`infra/`) — all Azure infrastructure as code

### Interaction Flow

```
User → Angular SPA → (Bearer token) → ASP.NET Core API → PostgreSQL
                                            ↓ publish
                                        RabbitMQ
                                            ↓ consume
                                    (future workers)
```

## Project Repository Structure

```
/workspace
├── api/                        # ASP.NET Core .NET 10 REST API
│   ├── src/
│   │   ├── EA.Api/             # Web API project (controllers, endpoints)
│   │   ├── EA.Domain/          # Domain models, interfaces, value objects
│   │   ├── EA.Infrastructure/  # EF Core DbContext, migrations, RabbitMQ integration
│   │   └── EA.Contracts/       # Shared DTOs, message contracts, JSON schemas
│   ├── tests/
│   │   ├── EA.Api.Tests/            # Unit tests
│   │   └── EA.Api.IntegrationTests/ # Testcontainers-based integration tests
│   ├── Dockerfile              # Multi-stage: SDK build → aspnet runtime
│   ├── Dockerfile.migrations   # EF Core migration bundle image
│   └── EA.sln
├── ui/                         # Angular 20 SPA
│   ├── src/
│   ├── Dockerfile              # Multi-stage: node build → nginx (local parity only)
│   ├── angular.json
│   └── package.json
├── infra/                      # Terraform (Azure)
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── backend.tf
│   └── modules/
│       ├── container-apps/
│       ├── postgres/
│       ├── static-web-app/
│       ├── container-registry/
│       ├── key-vault/
│       └── observability/
├── deploy/
│   ├── compose.yaml            # Full-stack local dev (API + Postgres + RabbitMQ + UI)
│   └── compose.override.yaml   # Dev overrides (hot reload, debug ports)
├── .github/
│   └── workflows/
│       ├── ci.yml              # Build, test, push images
│       ├── deploy.yml          # Terraform plan/apply
│       └── swa-deploy.yml      # Static Web Apps deploy
├── schemas/                    # JSON Schema message contracts (source of truth)
├── docs/                       # Architecture decisions, runbooks, API docs
└── CLAUDE.md                   # This file
```

## Technology Stack & Versions

| Layer | Technology | Version | Notes |
|---|---|---|---|
| API runtime | .NET | 10 | `mcr.microsoft.com/dotnet/sdk:10.0` / `aspnet:10.0` |
| API framework | ASP.NET Core | 10 | Minimal APIs preferred; controllers acceptable |
| ORM | EF Core + Npgsql | latest stable | Npgsql.EntityFrameworkCore.PostgreSQL |
| Messaging (.NET) | MassTransit | latest stable | RabbitMQ transport, outbox, sagas |
| Frontend | Angular | 20 | Standalone components, signals preferred |
| Frontend auth | MSAL Angular | latest | Auth code flow + PKCE via Entra ID |
| Database | PostgreSQL | 16 | Azure Flexible Server in cloud; `postgres:16` locally |
| Message broker | RabbitMQ | 4 | `rabbitmq:4-management` image |
| IaC | Terraform | ≥1.9 | AzureRM provider, azurerm backend |
| Containers | Docker | Compose v2 | Multi-stage builds, BuildKit |
| CI/CD | GitHub Actions | — | OIDC to Azure, Buildx, matrix builds |
| Observability | OpenTelemetry | — | Azure Monitor distro for .NET |

## Development Workflow

### Local Development (Docker-first)

```bash
# Start full stack
docker compose -f deploy/compose.yaml up --build

# API:        http://localhost:8000
# UI:         http://localhost:4200
# RabbitMQ:   http://localhost:15672 (guest/guest)
# Postgres:   localhost:5432 (demo/demo/demo)
```

### Running Tests

```bash
# API unit tests
dotnet test api/tests/EA.Api.Tests/

# API integration tests (requires Docker for Testcontainers)
dotnet test api/tests/EA.Api.IntegrationTests/

# UI tests
cd ui && npm test
```

### Database Migrations

```bash
# Add a migration
cd api/src/EA.Infrastructure
dotnet ef migrations add <MigrationName> -s ../EA.Api

# Apply locally
dotnet ef database update -s ../EA.Api

# Generate idempotent SQL script (for CI/CD)
dotnet ef migrations script --idempotent -s ../EA.Api -o ../../migrations.sql
```

## Coding Standards & Conventions

### General

- **No `// TODO` without a linked issue.** Use `// HACK:` only with justification.
- **All public APIs must have XML doc comments** (API project) or JSDoc (UI project).
- **Fail fast.** Validate inputs at boundaries; use guard clauses.
- **Prefer immutability.** Use `record` types in C#, `readonly` signals in Angular.

### C# / .NET

- Target `net10.0`. Enable nullable reference types (`<Nullable>enable</Nullable>`).
- Use file-scoped namespaces.
- Use primary constructors where appropriate.
- Use facade + repository design pattern, as called from appropriate REST API controllers.
- Follow ASP.NET Core conventions: `ProblemDetails` for errors (RFC 9457), `IResult` for minimal APIs.
- Logging: use `ILogger<T>` with structured logging (message templates, not string interpolation).
- EF Core: no lazy loading. Use explicit `.Include()` or projection queries.
- MassTransit consumers go in `EA.Infrastructure/Consumers/`.
- Migrations go in `EA.Infrastructure/Migrations/`.
- Connection strings and secrets come from configuration (environment variables in containers, Key Vault in Azure). **Never hardcode secrets.**

### Angular / TypeScript

- Use standalone components (no NgModules for feature components).
- Use Angular signals for local state; NgRx SignalStore for shared state.
- Use `inject()` function over constructor injection.
- HTTP calls go through dedicated service classes in `core/services/`.
- Use `environment.ts` for configuration; MSAL config in `auth/`.
- Strict TypeScript (`strict: true`). No `any` types without justification.

### Terraform

- Use modules for logical resource groups (see `infra/modules/`).
- All resources must be tagged: `environment`, `project`, `managed-by = "terraform"`.
- Use `terraform fmt` and `terraform validate` before committing.
- Variables must have `description` and `type`. Use `sensitive = true` for secrets.
- Remote state in Azure Blob Storage with locking.
- Prefer managed identities over passwords/keys everywhere.

### Docker

- All Dockerfiles use multi-stage builds.
- Copy dependency manifests first for layer caching (`*.csproj`, `package*.json`).
- Run as non-root user in production images.
- Pin base image versions (e.g., `dotnet/aspnet:10.0`, not `latest`).
- Use `.dockerignore` to exclude `bin/`, `obj/`, `node_modules/`, `.git/`.

### Messaging Contracts

- Message types live in `schemas/` as JSON Schema (Draft 2020-12).
- Routing keys are versioned: `analysis.job.requested.v1`.
- All messages must include: `messageId` (uuid), `correlationId` (uuid), `occurredAtUtc` (ISO 8601).
- Use the outbox pattern (MassTransit) for transactional consistency between DB writes and message publishing.

## API Design

- Resource-oriented URIs: `/api/v1/{resource}`.
- URL-path versioning (`/api/v1/...`).
- Consistent error responses using ProblemDetails.
- OpenAPI document generated via Scalar; keep it in sync.
- Health endpoints: `/health/live`, `/health/ready`, `/health/startup`.

## Observability

- OpenTelemetry SDK with Azure Monitor distro (`Azure.Monitor.OpenTelemetry.AspNetCore`).
- Structured logs to stdout/stderr (Container Apps routes to Log Analytics).
- Correlation IDs propagated across HTTP and RabbitMQ boundaries.
- Health probes wired to Container Apps liveness/readiness/startup checks.

## Deployment Pipeline

1. **PR** → CI builds, tests, lints (no push to ACR).
2. **Merge to `main`** → build images, tag with `sha-<short>`, push to ACR.
3. **Terraform plan** → saved as artifact, requires approval.
4. **Terraform apply** → updates Container Apps image tags, infra changes.
5. **Migration job** → Container Apps Job runs EF Core migration bundle.
6. **SWA deploy** → Angular build deployed to Static Web Apps.
7. **Smoke tests** → hit `/health/ready` + one end-to-end flow.

## Azure Resource Mapping

| Component | Azure Service | Terraform Resource |
|---|---|---|
| API | Container Apps | `azurerm_container_app` |
| RabbitMQ | Container Apps | `azurerm_container_app` |
| Migrations | Container Apps Jobs | `azurerm_container_app_job` |
| UI | Static Web Apps | `azurerm_static_web_app` |
| Database | PostgreSQL Flexible Server | `azurerm_postgresql_flexible_server` |
| Images | Container Registry | `azurerm_container_registry` |
| Secrets | Key Vault | `azurerm_key_vault` |
| Logs | Log Analytics | `azurerm_log_analytics_workspace` |
| APM | Application Insights | `azurerm_application_insights` |

## Agents

Specialized agents are available in `.claude/agents/` for focused work:

| Agent | File | Scope |
|---|---|---|
| Documentation | `docs.md` | Architecture docs, ADRs, runbooks, README updates |
| Testing | `testing.md` | Unit, integration, contract, and E2E tests |
| Frontend | `frontend.md` | Angular UI components, services, routing, auth |
| Backend | `backend.md` | ASP.NET Core API, EF Core, MassTransit, domain logic |
| Data Engine | `data-engine.md` | Python data engine, RabbitMQ consumers/producers, computation workflows |
| Database | `database.md` | EF Core migrations, schema design, query optimization |
| Infrastructure | `infrastructure.md` | Terraform, Docker, Compose, CI/CD workflows |
| Review | `review.md` | Code review, PR feedback, standards enforcement |

## Skills

Reusable workflow skills are available in `.claude/skills/` for focused scaffolding:

| Skill | Folder | Scope |
|---|---|---|
| Add EF Migration | `add-ef-migration/` | New EF Core migration with review artifacts |
| Add Integration Test | `add-integration-test/` | Testcontainers-based integration test for an endpoint or consumer |
| Add Message Contract | `add-message-contract/` | RabbitMQ message schema, .NET record, and MassTransit consumer |
| Add GitHub Workflow | `add-github-workflow/` | New GitHub Actions CI/CD workflow |
| Add Docker Service | `add-docker-service/` | New service in Docker Compose |
| Add Terraform Module | `add-terraform-module/` | New Terraform module for an Azure resource concern |
| Scaffold API Endpoint | `scaffold-api-endpoint/` | New REST endpoint with domain entity, EF config, DTOs, and migration |
| Scaffold Angular Feature | `scaffold-angular-feature/` | New Angular feature with component, service, and route |
| Scaffold Data Engine Worker | `scaffold-data-engine-worker/` | New Python RabbitMQ consumer, Pydantic model, workflow, and test stubs |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/awpala) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-12 -->
