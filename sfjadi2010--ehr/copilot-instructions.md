## ehr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Full-stack Electronic Health Record (EHR) system. Backend: ASP.NET Core 9 Web API with Clean Architecture + CQRS. Frontend: React 19 + TypeScript + Vite. Database: PostgreSQL 16 via EF Core.

## Commands

### Backend

```bash
# Start only the database (required for local API dev)
docker compose up postgres pgadmin -d

# Run the API locally (from repo root or src/EHR.API)
cd src/EHR.API && dotnet run
# API: http://localhost:5000 | Scalar docs: http://localhost:5000/scalar

# Add a new EF Core migration (run from repo root)
cd src/EHR.Infrastructure
dotnet ef migrations add <MigrationName> --startup-project ../EHR.API --output-dir Persistence/Migrations

# Apply migrations
dotnet ef database update --startup-project ../EHR.API
```

### Frontend

```bash
cd client
npm install        # first time or after package changes
npm run dev        # http://localhost:5173
npm run build      # production build (tsc -b && vite build)
npm run lint       # ESLint
```

### Docker (full stack)

```bash
docker compose up --build     # build and start all services
docker compose down           # stop and remove containers
```

Services: API → `localhost:5000`, Frontend → `localhost:5173`, pgAdmin → `localhost:5050` (admin@ehr.local / admin123)

## Architecture

### Backend layers (`src/`)

```
EHR.API           → Controllers, middleware, DI wiring, Program.cs
EHR.Application   → CQRS handlers, validators, DTOs, interfaces
EHR.Domain        → Entities, enums, value objects (no dependencies)
EHR.Infrastructure → EF Core DbContext, repositories, JWT, seeding
EHR.Shared        → Result<T>, PagedResult<T>, ApiResponse<T> — shared across all layers
```

**Dependency flow:** API → Application → Domain ← Infrastructure

### CQRS pattern

Every write is a `Command`, every read is a `Query`, both dispatched via MediatR from controllers. All handlers live in `EHR.Application/Features/<Module>/`. Example structure:

```
Features/Patients/
  Commands/CreatePatient/CreatePatientCommand.cs + Handler + Validator
  Queries/GetPatients/GetPatientsQuery.cs + Handler
  DTOs/PatientDto.cs
```

Controllers inherit `BaseController` which exposes `Send(command)` and `HandleResult(result)` — the latter maps `Result<T>` to the appropriate HTTP response.

### Validation pipeline

`ValidationBehavior<TRequest, TResponse>` (registered as MediatR pipeline behavior) auto-runs FluentValidation before every handler. Validation failures surface as `400 Bad Request` via `GlobalExceptionMiddleware`.

### Result pattern

Handlers return `Result<T>` (from `EHR.Shared`). Never throw for business logic failures — use `Result<T>.Failure("message")`. `HandleResult()` in `BaseController` converts this to `404`/`400`/`200` automatically.

### Soft deletes

All entities inherit `SoftDeleteEntity` which adds `DeletedAt`. `SaveChangesAsync` on `EhrDbContext` automatically sets `UpdatedAt`. Deleted records are excluded via EF global query filters.

### NuGet versions

Managed centrally in `Directory.Packages.props` at the repo root (Central Package Management). **Never add `Version=` to `<PackageReference>` in `.csproj` files** — add/update versions only in `Directory.Packages.props`.

### Frontend structure (`client/src/`)

```
api/          → Typed Axios functions per module (patientsApi.ts, etc.)
features/     → Module folders: hooks/, pages/, types/, schemas/, components/
stores/       → Zustand: authStore (JWT + user), uiStore
components/   → ui/ (Radix/shadcn primitives), shared/ (DataTable, SearchInput, Pagination)
lib/          → axios.ts (interceptors + auto-refresh), utils.ts
router/       → React Router v7 routes + PrivateRoute guard
```

**Data fetching:** TanStack Query (`useQuery`/`useMutation`) in feature hooks. Each module has a `use<Module>.ts` hook that wraps the API calls and exposes query keys for cache invalidation.

**Auth flow:** Login → JWT stored in Zustand `authStore` → Axios request interceptor attaches `Bearer` token → 401 response triggers silent token refresh (with a queue to prevent concurrent refresh races) → refresh failure clears auth and redirects to login.

### Docker / Dockerfile notes

The API `Dockerfile` (`src/EHR.API/Dockerfile`) copies `Directory.Packages.props` before the individual `.csproj` files so that `dotnet restore` has CPM context. If you add new projects, add them to the COPY block before the `RUN dotnet restore` line.

## Default credentials (seeded)

| Role | Email | Password |
|------|-------|----------|
| Physician | dr.smith@ehr.local | Doctor@123! |
| Admin | admin@ehr.local | Admin@123! |

## Key config

- **JWT:** `appsettings.json` → `Jwt.SecretKey` (32+ chars), 15-min access token, 7-day refresh token stored on `ApplicationUser`
- **DB connection:** `appsettings.json` → `ConnectionStrings.DefaultConnection` (local) or env var `ConnectionStrings__DefaultConnection` (Docker)
- **CORS:** `appsettings.json` → `Cors.AllowedOrigins`
- **Identity tables:** prefixed `ehr_` (e.g. `ehr_users`, `ehr_roles`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sfjadi2010) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
