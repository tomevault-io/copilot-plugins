## yawn

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Yawn is a data workflow platform with a polyglot architecture:

- **Backend**: Go with Domain-Driven Design (DDD) using Echo framework
- **Frontend**: React + TypeScript with Vite (in `ui/` directory)
- **Python SDK**: Asset and pipeline definitions (in `library/` directory)
- **Shared Types**: TypeScript generated from Go structs for type safety

## Development Commands

### Backend (Go API)

**Testing:**

```bash
# Run all tests
go test ./...

# Run tests with verbose output
go test -v ./...

# Run specific test file
go test -v ./internal/domain/services

# Run single test
go test -v ./internal/domain/services -run TestProjectService_Create

# Generate tests for a file (as specified in tech-stack.md)
gotests -all -use_go_cmp -w -parallel <file_name>.go
```

**Building & Running:**

```bash
# Build the application
go build ./...

# Run the server
go run main.go serve

# Run with hot reload (using air)
air
```

**Code Quality:**

```bash
# Format code
gofmt -s -w .
goimports -w .

# Run linter
golangci-lint run

# Run linter on specific files
golangci-lint run ./internal/domain/services ./internal/interfaces/handlers
```

**Dependencies:**

```bash
# Tidy modules
go mod tidy

# Download dependencies
go mod download
```

### Frontend (React UI)

**Install UI components (Shadcn):**

```bash
pnpx shadcn@latest add <component_name>
```

## Architecture

### Backend Domain-Driven Design

The Go backend follows DDD principles with clear layer separation:

**Domain Layer** (`internal/domain/`):

- **Models**: Core entities (User, Project, etc.) with GORM annotations
- **Services**: Business logic and validation (e.g., ProjectService, UserService)
- **Repositories**: Data access interfaces implemented by infrastructure layer

**Infrastructure Layer** (`internal/infrastructure/`):

- **Database**: GORM setup with SQLite (dev) / PostgreSQL (prod)
- **Web**: Echo framework setup and middleware
- **Logger**: Zerolog structured logging

**Interface Layer** (`internal/interfaces/`):

- **Handlers**: HTTP request handlers using Echo framework
- **DTOs**: Request/response data transfer objects with validation

**App Layer** (`internal/app/`):

- **Dependency Injection**: Uses uber-go/fx for compile-time DI
- **Bootstrap**: Application startup and lifecycle management

### Key Architecture Patterns

**Dependency Injection**: All dependencies are wired through fx providers in `app.go`. New services, repositories, or handlers must be registered as providers.

**Authentication & Authorization**:

- JWT-based authentication with access/refresh tokens
- Role-based access control (RBAC) for system-level permissions
- Project-level role system (Owner, Maintainer, Viewer) for resource permissions

**Database**:

- GORM ORM with UUID v7 primary keys
- Soft deletes using GORM's `DeletedAt` field
- Multi-environment support (SQLite for dev, PostgreSQL for prod)

**Testing Strategy**:

- Table-driven tests with comprehensive scenarios
- Mock implementations using testify/mock for repository interfaces
- Business logic testing focus (75% coverage requirement)
- Deterministic UUIDs in tests to avoid flakiness

### Frontend Architecture

- **Framework**: React 18+ with functional components and hooks
- **Routing**: TanStack Router with file-based routing
- **State Management**: Zustand for simple, lightweight state management
- **UI Components**: Shadcn component library with Tailwind CSS
- **Build**: Vite for fast development and optimized builds
- **Type Safety**: Generated TypeScript interfaces from Go structs ensure backend-frontend consistency

### Integration Points

**Type Generation**: TypeScript interfaces are generated from Go model structs to ensure type safety across the frontend-backend boundary.

**Build Process**: In production, the React frontend is embedded in the Go binary using Go's `embed` directive, creating a single executable.

## Development Guidelines

**Code Standards**:

- Follow Uber Go Style Guide
- Use structured logging with zerolog
- Implement explicit error returns with proper error wrapping
- Maintain 75% test coverage with focus on business logic

**MCP Integration**: Use context7, browsermcp, mcp-gopls, and serena for code maintenance, documentation lookup, and debugging assistance.

**Project Structure**: Respect domain boundaries - business logic stays in services, data access in repositories, HTTP concerns in handlers.

## Common Development Patterns

**Adding New Features**:

1. Define/update models in `internal/domain/models/`
2. Create repository interface and implementation
3. Implement service layer with business logic
4. Add handler with Echo framework
5. Create DTOs for request/response
6. Wire dependencies in `app.go`
7. Add comprehensive tests
8. Update routing configuration

**Error Handling**: Use structured error returns with proper context. Service layer returns business errors, handlers translate to HTTP status codes.

**Testing Approach**: Write table-driven tests covering success cases, validation errors, and edge cases. Use deterministic UUIDs and proper mock setup for reliable tests.
- Always use context7 to search for library docs, use serena to find file/code, browsermcp to use the browser for either testing or searching things
- only use bun and bunx, no pnpm, pnpx, npm, npx or even yarn, we're only using bun, nothing else
- never run dev mode for api or ui by yourself, ask for user to turn it on

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ditwrd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
