## backer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Backer is a distributed backup/file synchronization system supporting multiple workstations, cloud providers (OneDrive, Dropbox, Google Drive), local NAS devices, and Nextcloud. The system uses rclone as the underlying file transfer engine.

## Build Commands

```bash
# Build the entire solution
dotnet build Backer.sln

# Build for release
dotnet build Backer.sln -c Release

# Run all tests (see docs/TESTING.md)
dotnet test Backer.sln

# Run unit tests only - no database needed
dotnet test tests/Hannibal.Tests/ tests/WorkerRClone.Tests/ tests/Tools.Tests/

# Run a single test
dotnet test tests/Hannibal.Tests/ --filter "FullyQualifiedName~TestMethodName"

# Apply database migrations (from Hannibal project)
dotnet ef database update --project application/Hannibal/

# Run individual projects
dotnet run --project Api/                    # Web API
dotnet run --project BackerAgent/            # Background service
dotnet run --project frontend/Poe/           # Blazor web UI

# Docker deployment
docker-compose up
```

## Technology Stack

- **.NET 9.0** (SDK 9.0.308 - see global.json)
- **Blazor Server** for web frontend (Poe)
- **Avalonia** for cross-platform desktop (YourBacker)
- **Entity Framework Core 9.0** with PostgreSQL
- **SignalR** for real-time communication
- **xUnit + FluentAssertions** for testing

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    FRONTEND LAYER                       │
│  Poe (Blazor Web UI)       YourBacker (Avalonia Desktop)│
└──────────────────────────┬──────────────────────────────┘
                           │ HTTP/SignalR
                           ▼
┌─────────────────────────────────────────────────────────┐
│                   API LAYER (Api/)                      │
│  - ASP.NET Core Web API                                 │
│  - JWT authentication                                   │
│  - RuleScheduler (event-driven job creation)            │
│  - Core business logic in application/Hannibal/         │
└──────────────────────────┬──────────────────────────────┘
                           │ HTTP/SignalR
                           ▼
┌─────────────────────────────────────────────────────────┐
│               WORKER LAYER (BackerAgent/)               │
│  - Windows Service / Linux Systemd                      │
│  - WorkerRClone (worker/WorkerRClone/)                  │
│  - RCloneStateMachine controls service lifecycle        │
│  - Storage providers for OAuth2 & local storage         │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   rclone     │ (external CLI tool)
                    └──────────────┘
```

## Key Projects

| Project | Path | Purpose |
|---------|------|---------|
| Api | `Api/` | ASP.NET Core Web API with JWT auth, Swagger docs |
| Hannibal | `application/Hannibal/` | Core library: services, EF Core models, scheduling |
| BackerAgent | `BackerAgent/` | Background service host (Windows/Linux) |
| WorkerRClone | `worker/WorkerRClone/` | RClone wrapper, state machine, storage providers |
| Poe | `frontend/Poe/` | Blazor web frontend |
| YourBacker | `YourBacker/` | Avalonia cross-platform desktop control app |
| Tools | `Tools/` | Shared utilities (auth handlers, token services) |
| Hannibal.Tests | `tests/Hannibal.Tests/` | Unit tests: scheduling, path overlap, OAuth2 client |
| WorkerRClone.Tests | `tests/WorkerRClone.Tests/` | Unit tests: password obscurer, Graph provider |
| Tools.Tests | `tests/Tools.Tests/` | Unit tests: HTTP auth handlers |
| Hannibal.IntegrationTests | `tests/Hannibal.IntegrationTests/` | API + DB integration (needs PostgreSQL) |

## Key Domain Concepts

- **Rule**: Defines what to backup (source/destination endpoints, schedule)
- **Job**: A single execution instance of a Rule
- **Storage**: Cloud or local storage configuration (credentials, provider type)
- **Endpoint**: A specific path within a Storage
- **RuleScheduler**: Event-driven scheduler that creates Jobs from Rules

## Authentication

JWT-based authentication with auto-refresh:
- API issues tokens via `/api/authb/v1/token`
- `AutoAuthHandler` in BackerAgent intercepts 401s and auto-refreshes
- Frontend uses cookie auth with `HttpContextTokenProvider` for API calls
- JWT config in `appsettings.json` under `Jwt:Key`, `Jwt:Issuer`, `Jwt:Audience`

## Storage Providers

Located in `worker/WorkerRClone/Services/Providers/`:
- **OAuth2**: OneDrive, Dropbox, Google Drive
- **Local**: SMB, Local filesystem, Nextcloud

## Configuration

- `appsettings.json` - base configuration
- `appsettings.{Environment}.json` - environment overrides
- `appsettings.{MachineName}.json` - machine-specific
- Database connection string: `ConnectionStrings:DefaultConnection`, else the
  `HANNIBAL_DB_CONNECTION` environment variable (what the live deployment uses),
  else a localhost fallback
- `Hannibal:SkipStartupMigration` - skip migrate/seed at startup (default false)
- `OAuth2:RedirectUri` - OAuth2 callback, defaults to `http://localhost:53682/`
  (the BackerAgent's local listener)
- BackerAgent credentials: `RCloneService:BackerUsername/BackerPassword`

## Testing

See `docs/TESTING.md`. Four test projects; `tests/Hannibal.IntegrationTests`
needs a local PostgreSQL and skips cleanly without one. No test touches live
backup data or starts rclone.

---
> Source: [tweggen/backer](https://github.com/tweggen/backer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
