## wordmesh

> This file provides guidance to Claude Code (claude.ai/claude-code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/claude-code) when working with code in this repository.

## Project Overview

WordMesh is a personal knowledge network builder where users connect words and concepts to build their semantic world. It's a bilingual web application (Chinese/English) using Rust backend with Next.js TypeScript frontend.

**Tech Stack:**
- **Backend:** Rust + Actix Web + PostgreSQL + Neo4j
- **Frontend:** Next.js 16 + React 19 + TypeScript + Tailwind CSS + shadcn/ui
- **Databases:** PostgreSQL (relational data) + Neo4j (graph relationships)

---

## Development Commands

### Backend (Rust)

```bash
cd WordMesh-backend

# Development server
cargo run

# Build
cargo build

# Run tests
cargo test

# Run specific test
cargo test -- test_name

# Code quality checks
cargo check
cargo clippy
cargo fmt
```

### Frontend (Next.js)

```bash
cd wordmesh-frontend

# Development server
npm run dev
# or
pnpm dev

# Build for production
npm run build

# Lint
npm run lint
```

### Database Services

```bash
cd deployment
docker compose up -d     # Start PostgreSQL and Neo4j
docker compose down        # Stop services
```

**Database URLs:**
- PostgreSQL: `localhost:5432` (user: wordmesh, password: wordmesh123, db: wordmesh_dev)
- Neo4j Browser: `http://localhost:7474` (neo4j/wordmesh123)
- Backend API: `http://localhost:8080`
- Frontend: `http://localhost:3000`

---

## Architecture Overview

### Backend Architecture (Rust)

The backend follows a clean layered architecture with clear separation of concerns:

```
src/
├── main.rs           # Application entry, server configuration
├── config/           # Multi-environment config (dev/test/prod)
├── controller/       # HTTP interface layer (AuthController, WordController, AssocController)
├── service/          # Business logic layer (AuthService, WordService, SenseService, AssocService)
├── repository/       # Data access layer (PgUserRepository, PgWordRepository, Neo4jGraphRepository)
├── domain/           # Domain models with business rules (User, UserWord, UserSense, CanonicalKey)
├── dto/              # Request/Response data transfer objects
├── middleware/       # AuthGuard, RequestId middleware
└── util/             # Error handling, response builder, validation
```

**Key Patterns:**
- **Controller Pattern:** Each controller has a `configure()` method that registers routes and wraps protected endpoints with `AuthGuard`
- **Repository Pattern:** Trait-based repositories (`UserRepository`, `WordRepository`, `GraphRepository`) with PostgreSQL and Neo4j implementations
- **Unified Response:** All endpoints return `ApiResponse<T>` with structure `{code, message, data, traceId, timestamp}`
- **Error Hierarchy:** `AppError` wraps domain errors (BusinessError, DbError, AuthError, etc.) with specific error codes (Auth: 401x, Word: 420x, Link: 430x)
- **Request Tracing:** `RequestId` middleware injects unique traceId for log correlation

**Domain Model Key Concepts:**
- `CanonicalKey`: Value object for normalized text keys (case-insensitive, whitespace-trimmed)
- `UserWord`: User's personal word entry with tags, notes, and senses
- `UserSense`: User's definition/understanding of a word (can be primary)
- Entities enforce business rules (e.g., no duplicate sense texts, only one primary sense)

### Frontend Architecture (Next.js)

The frontend uses a modular feature-based architecture:

```
src/
├── app/(app)/        # Next.js App Router pages (login, register, words, settings)
├── modules/          # Feature modules (auth, word, sense, association, network)
│   ├── [feature]/
│   │   ├── components/   # Feature-specific UI components
│   │   ├── api/          # Feature API client functions
│   │   └── index.ts      # Public exports
├── shared/            # Cross-cutting utilities
│   ├── api/            # Unified API client (TokenStorage, error handling)
│   ├── constants/      # API and app constants
│   ├── types/          # Shared TypeScript types
│   └── utils/          # Error handlers, toast, validation, format
├── components/ui/     # shadcn/ui components
├── hooks/             # Custom React hooks
└── lib/               # Utilities (cn for className merging)
```

**Key Patterns:**
- **API Client:** Centralized `apiClient` object with automatic auth token injection, timeout handling, and error parsing
- **Token Management:** `TokenStorage` manages access/refresh tokens in localStorage
- **Error Handling:** Unified error parsing with user-friendly Chinese/English messages via `toast` notifications
- **Feature Modules:** Each domain feature (word, sense, association) exports its components, API functions, and types via `index.ts`

---

## API Conventions

### Unified Response Format

All endpoints return HTTP 200 with a JSON envelope:

```json
{
  "code": 2000,           // 2000 = success, 4xxx = business errors, 401x = auth, 420x = word, 430x = link
  "message": "OK",
  "data": { ... },         // null for errors
  "traceId": "uuid",
  "timestamp": 1234567890
}
```

### Authentication

- Public endpoints: `POST /api/v1/auth/register`, `POST /api/v1/auth/login`
- Protected endpoints require `Authorization: Bearer <access_token>` header
- Access tokens expire (default 60s), refresh tokens extend sessions (default 120s)

### Route Naming

- `/api/v1/health` - Health check
- `/api/v1/auth/*` - Authentication (register, login, refresh, profile)
- `/api/v1/words/my` - Manage user's personal words
- `/api/v1/words/my/senses` - Manage word senses
- `/api/v1/words/associations` - Create/delete associations

---

## Code Style Guidelines

### Backend (Rust)

- **Error Handling:** Always return `Result<T, AppError>` from functions that can fail
- **Immutability:** Default to immutable bindings (`let`), use `mut` only when necessary
- **Validation:** Input validation in DTO layer with `validator` crate; business rules in domain layer
- **No SQL Injection:** Always use parameterized queries via SQLx macros
- **Logging:** Use `tracing::{info, warn, error, debug}` with structured fields
- **Tests:** Include unit tests in same file with `#[cfg(test)]` module

### Frontend (TypeScript)

- **No Mutations:** Use spread operators for immutable updates (`{ ...item, field: value }`)
- **Error Boundaries:** Wrap API calls in try/catch, show user-friendly error messages
- **Type Safety:** Avoid `any`; use proper types from `shared/types/`
- **Component Size:** Keep components focused; extract utilities/helpers to separate files
- **No console.log:** Use proper error handling and toast notifications instead

---

## Important Notes

- **Dual Database:** PostgreSQL stores structured data (users, words, senses), Neo4j stores graph relationships (associations)
- **Text Normalization:** `canonicalize()` utility lowercases and trims text for case-insensitive comparison
- **Bilingual:** UI and error messages support Chinese and English
- **Development Status:** Auth module is complete (100%), word module partially implemented (~60%), association module in progress (~40%)
- **Missing Routes:** Word and Association controllers exist but routes are not fully implemented in `main.rs`

---

## Testing

### Backend Tests

```bash
# Run all tests
cargo test

# Run tests with output
cargo test -- --nocapture

# Run specific module tests
cargo test --package Wordmesh-backend --lib domain::word::tests
```

Test coverage: Auth module has comprehensive unit/integration tests. Word and association modules need test coverage.

### Frontend Tests

(E2E tests with Playwright are planned but not yet implemented)

---

## Configuration

Backend uses TOML config files in `WordMesh-backend/config/`:
- `default.toml` - Base configuration
- `development.toml` - Local development overrides
- `testing.toml` - Test environment
- `production.toml` - Production settings

Sensitive values (JWT secrets, DB passwords) read from environment variables or Docker secrets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CleverOnion) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
