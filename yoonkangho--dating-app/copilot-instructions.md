## dating-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

The primary focus of this project is
- to learn how to better use AI in software development
- to get hands-on experience in establishing and maintaining simple, production-grade service

## Documentation Reference

| Document | Purpose |
|----------|---------|
| `PROJECT-CHARTER.md` | Product requirements and user flows |
| `ERD.md` | Database schema, tables, relationships, indexes |
| `STATE-MACHINES.md` | State transitions, business rules, side effects |
| `skills/BACKEND.md` | Kotlin + Spring Boot patterns |
| `skills/DATABASE.md` | PostgreSQL + JPA patterns |
| `skills/TESTING.md` | Testing patterns |

**Before implementing code, read the relevant documents above.**

## Project Overview

**Guaranteed Meeting Dating App** — A dating app where every match leads to a real meeting. Quality over quantity through curated membership and structured outcomes.

### Core Differentiators

1. **Curated membership** — Admin approval gates entry (identity verification + manual review)
2. **Asymmetric discovery** — Women browse and initiate; men only see women who've expressed interest
3. **Structured outcomes** — Match → Schedule → Meet → Feedback. No endless chat.
4. **Reputation signals** — User behavior (responsiveness, reliability) tracked and visible

## Build Commands

```bash
# Build the project
./gradlew build

# Run the application
./gradlew bootRun

# Run all tests
./gradlew test

# Run a specific test class
./gradlew test --tests ClassName

# Run a specific test method
./gradlew test --tests "ClassName.methodName"

# Clean build artifacts
./gradlew clean
```

## Development Setup

Start required services with Docker Compose before running the application:
```bash
docker-compose up -d   # Starts PostgreSQL (port 5432) and Redis (port 6379)
./gradlew bootRun      # Run with DevTools hot reload
```

## Architecture

**Spring Boot Kotlin application using Spring Modulith** for modular monolith architecture.

### Technology Stack

- Kotlin 2.3.0 / Java 25 / Spring Boot 4.0.1
- PostgreSQL (persistence) with Flyway migrations
- Redis (caching)
- Spring Security (authentication/authorization)
- Spring Data JPA (data access)
- Spring Modulith (module boundaries)

### Module Structure

```
com.kenneth.datingapp
├── user/          # User accounts, profiles, photos, approval requests
├── matching/      # Ok flow, meetings, proposals, scheduling
├── feedback/      # Post-meeting feedback, ratings, disputes
└── admin/         # Admin users, actions, warnings
```

Each module has:
- Public API in package root (entities, services, DTOs, events)
- Internal implementation in `internal/` subpackage (repositories, helpers)

Spring Modulith enforces boundaries — modules communicate through public APIs and domain events.

## Domain Model Summary

> **Full details in STATE-MACHINES.md**

### User Status Flow

```
VERIFICATION_PENDING → WAITLISTED → REVIEW_PENDING → ACTIVE
                                                   ↘ REJECTED (admin can reactivate)
                                    ACTIVE → BANNED (admin can unban)
```

### Ok Status Flow

```
PENDING → ACCEPTED (creates Meeting)
        ↘ DECLINED / EXPIRED / UNDONE (all terminal, blocks retry with same pair)
ACCEPTED → UNDONE (only before Meeting reaches SCHEDULED)
```

### Meeting Status Flow

```
PENDING_PROPOSAL → PENDING_FINALIZATION → SCHEDULED → COMPLETED
                                                    ↘ CANCELLED
SCHEDULED → PENDING_PROPOSAL (reschedule, max 1)
```

### Key Business Rules

| Rule | Description |
|------|-------------|
| One ok per pair | Woman can only send one ok to a specific man ever |
| Max 3 active oks | Woman can have at most 3 oks in PENDING/ACCEPTED with active meetings |
| 7-day ok expiration | Man must respond within 7 days |
| 2-day proposal deadline | Woman must propose dates within 2 days of mutual ok |
| 2-day finalization deadline | Man must select date within 2 days of proposal |
| 3-week meeting deadline | All proposed dates must be within 3 weeks of mutual ok |
| Max 1 reschedule | Either party can request; goes back to PENDING_PROPOSAL |
| Mandatory feedback | User locked out until feedback submitted (3h after meeting) |
| Contact exchange | Only if meeting happened; one-sided (your phone sent if you say yes) |

### Reputation Flags (visible on profile)

- `has_undone_ok` — User withdrew before meeting was scheduled
- `has_let_ok_expire` — Man didn't respond within 7 days

## Coding Conventions

### Kotlin Style

- Use data classes for DTOs and value objects
- Prefer immutability (`val` over `var`)
- Use sealed classes for result types
- Extension functions for utility operations
- Business logic methods on entities (e.g., `user.approve()`, `ok.accept()`)

### Naming Conventions

- Entities: singular nouns (`User`, `Ok`, `Meeting`)
- Repositories: `<Entity>Repository` (in `internal/` package)
- Services: `<Domain>Service` (e.g., `MatchingService`, `MeetingService`)
- Controllers: `<Domain>Controller`
- DTOs: `<Action><Entity>Request/Response`
- Events: `<Entity><Action>Event` (e.g., `OkAcceptedEvent`)

### Database Conventions

- Table names: snake_case, plural (`users`, `oks`, `meetings`)
- Column names: snake_case (`created_at`, `user_id`)
- Foreign keys: `<referenced_table_singular>_id`
- Indexes: `idx_<table>_<columns>`
- Migrations: `V<version>__<description>.sql`

### Module Communication

Modules communicate through public APIs and domain events. See `skills/BACKEND.md` for implementation patterns.

## Testing

See `skills/TESTING.md` for testing patterns, conventions, and examples.

## Open Technical Decisions

| Decision | Status | Notes |
|----------|--------|-------|
| Identity verification provider | TBD | Design for abstraction (interface + adapter) |
| Push notification service | TBD | FCM, APNs, or unified service |
| Image/photo storage | TBD | S3, GCS, or similar |
| Frontend stack | TBD | Will affect API design (mobile-first) |

---
> Source: [yoonkangho/dating-app](https://github.com/yoonkangho/dating-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
