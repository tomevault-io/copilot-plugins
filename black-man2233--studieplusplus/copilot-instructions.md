## studieplusplus

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

StudiePlusPlus is a full-stack school management platform with:
- **Frontend**: Ionic + Vue 3 + TypeScript (mobile-first, cross-platform via Capacitor)
- **Backend**: ASP.NET Core 8.0 following Clean Architecture
- **Database**: SQL Server via Entity Framework Core (code-first)
- **Deployment**: Docker Compose

The UI strings are in Danish (e.g., "Hjem", "Lektier", "Skema").

## Commands

### Frontend (`src/client-app/studieplusplusgui/`)

```bash
npm install
npm run dev          # Vite dev server
npm run build        # TypeScript + Vite production build
npm run lint         # ESLint
npm run test:unit    # Vitest unit tests
npm run test:e2e     # Cypress E2E tests
```

### Backend (`src/server-api/`)

```bash
dotnet restore
dotnet build
dotnet run --project StudiePlusPlus.API
dotnet test          # Run all tests
```

### Full Stack (Docker)

```bash
docker-compose up --build   # Starts API on ports 8080/8081
```

API docs available at `/scalar/` once running.

## Architecture

### Backend — Clean Architecture

The backend has four projects with strict dependency rules (outer layers depend on inner):

1. **Domain** (`StudiePlusPlus.Domain/`) — Core entities with no dependencies
   - Entities: `User` (base), `Student`, `Teacher`, `Class`, `Subject`, `Grade`, `Enrollment`, `WeeklySchedule`
   - `ValueObjects/` for concepts like `Email`

2. **Application** (`StudiePlusPlus.Application/`) — Business logic, depends only on Domain
   - Organized by feature folder: `Features/Students/`, `Features/Teachers/`, etc.
   - `Abstractions/` contains all interfaces (repositories, mappers, security)
   - `Common/Handlers/` provides generic `ReadHandler` and `WriteHandler` to reduce CRUD boilerplate

3. **Infrastructure** (`StudiePlusPlus.Infrastructure/`) — EF Core, repositories, auth; implements Application interfaces
   - `Persistence/AppDbContext.cs` — EF DbContext
   - `Persistence/UnitOfWork.cs` — Unit of Work pattern wrapping all repositories
   - Repository implementations in `Persistence/Repositories/`

4. **API** (`StudiePlusPlus.API/`) — ASP.NET Core controllers; depends on Application
   - `CrudController.cs` is a generic base controller providing standard CRUD endpoints
   - Resource-specific controllers inherit from it: `StudentsController`, `TeachersController`, etc.

### Frontend — Tab-Based Vue 3 + Ionic

- **Router** (`src/router/`): Tab-based navigation with `TabsPage` as the main shell
- **Views** (`src/views/`): Full-page components — `HomePage`, `SchedulePage`, `MessagePage`, `ProfilePage`, `SettingsPage`
- **Components** (`src/components/`): Reusable Ionic-wrapped Vue components

### REST API Pattern

All resources follow the generic CRUD pattern from `CrudController`:
- `GET /api/{resource}` — list
- `GET /api/{resource}/{id}` — get one
- `POST /api/{resource}` — create
- `PUT /api/{resource}/{id}` — update
- `DELETE /api/{resource}/{id}` — delete

Available resources: `classes`, `students`, `teachers`, `subjects`, `grades`, `enrollments`, `weekly-schedules`, `auth`.

## Key Conventions

- **Strict nullability**: The backend has `<Nullable>enable</Nullable>` and `<TreatWarningsAsErrors>true</TreatWarningsAsErrors>` — all nullable warnings are compile errors.
- **Generic CRUD**: Before adding per-resource CRUD code, check if `CrudController` / `ReadHandler` / `WriteHandler` covers the use case.
- **Feature folders**: New Application layer features go in `Application/Features/{FeatureName}/` with DTOs and mappers alongside the feature code.
- **Repository interfaces live in Application**: Implementations in Infrastructure; never reference Infrastructure from Application or Domain.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/black-man2233) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
