## backend

> Backend architecture and implementation standards


# Backend Engineering Standards

You are a senior backend engineer specialized in:
- ASP.NET Core
- Distributed systems
- Event-driven architectures
- High performance APIs
- Observability
- Snapshot processing systems

# Architecture Rules

The project MUST follow:
- Clean Architecture
- SOLID principles
- CQRS when beneficial
- Repository Pattern
- Dependency Injection
- Immutable historical storage

Layers:
- Domain
- Application
- Infrastructure
- API

Never:
- place business logic inside controllers
- access Infrastructure directly from API
- bypass Application layer
- create circular dependencies

# API Rules

Controllers must:
- be thin
- only orchestrate requests
- never contain calculations
- never contain ranking logic

Use:
- DTOs
- validation
- async endpoints
- CancellationToken

All endpoints must:
- return proper status codes
- support tracing
- support structured logging

# Database Rules

Use:
- Entity Framework Core
- PostgreSQL

Database principles:
- append-only history when possible
- immutable snapshots
- normalized critical data
- indexed ranking queries

Always:
- configure entities explicitly
- create indexes for ranking fields
- use migrations
- avoid lazy loading

Never:
- use SELECT *
- load unnecessary collections
- perform N+1 queries

# Snapshot Processing Rules

The arena API is snapshot-based.

The backend must:
- ingest snapshots periodically
- preserve raw snapshots
- compare historical states
- reconstruct inferred events

Snapshots are the source of truth.

Detected events are derived data.

# Scheduling Rules

Arena snapshot ingestion is schedule-based.

Valid arena times:
- 13:00
- 19:00
- 20:30
- 23:00

Official collection times:
- 13:31
- 19:31
- 21:01
- 23:31

Timezone:
- America/Sao_Paulo

Thursday Rule:
- Thursday 13:00 arena NEVER exists
- snapshot collection at Thursday 13:31 must NEVER execute

The system must:
- execute only during valid windows
- avoid continuous polling
- prevent duplicate snapshot ingestion
- persist official arena timestamps

Workers must:
- be timezone-aware
- validate execution windows
- skip invalid schedules safely

Never:
- use machine local timezone
- continuously poll the external API
- generate redundant snapshots

# Event Reconstruction Rules

The system must infer:
- new wins
- likely winning teams
- ranking progression
- reward eligibility

Detection rules must:
- be deterministic
- be traceable
- generate confidence scores

Never:
- overwrite original snapshots
- mutate historical records

# Worker Rules

Background workers must:
- be idempotent
- support retries
- support cancellation
- generate logs
- be independently testable

Required workers:
- Snapshot Worker
- Diff Worker
- Event Reconstruction Worker
- Leaderboard Worker

# Logging Rules

Use:
- Serilog
- structured logs
- correlation ids
- execution timing

Every critical operation must log:
- operation name
- execution duration
- success/failure
- exception details
- snapshot identifiers

# Observability Rules

Implement:
- OpenTelemetry
- health checks
- tracing
- metrics

The system must support:
- replay analysis
- debugging
- historical reconstruction
- root cause analysis

# Caching Rules

Use Redis for:
- leaderboard cache
- latest winners
- hot ranking queries

Cache invalidation must:
- be deterministic
- occur after snapshot processing
- avoid stale rankings

# Performance Rules

Optimize:
- ranking calculations
- historical lookups
- snapshot diffing
- leaderboard queries

Prioritize:
- batching
- compiled queries
- async processing
- pagination

# Testing Rules

The backend must include:
- unit tests
- integration tests
- domain validation tests

Business rules must always be tested.

Especially:
- Combat Score calculations
- victory detection
- snapshot diffing
- winner inference
- ranking generation
- schedule validation
- timezone validation

# Code Quality Rules

Code must:
- compile successfully
- avoid dead code
- avoid duplicated logic
- use strong typing
- be production-ready

Prefer:
- small services
- explicit naming
- deterministic behavior
- pure domain logic

Never:
- create fake implementations
- ignore exceptions
- silently swallow failures
- introduce hidden side effects

---
> Source: [davidzaque-leal/royal-arena](https://github.com/davidzaque-leal/royal-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
