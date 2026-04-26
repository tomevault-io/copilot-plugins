## ats-hr-resume

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go-based resume management microservice for an ATS (Applicant Tracking System) platform. It handles resume uploads, parsing, storage, and event publishing.

## Commands

```bash
# Run the service
go run ./cmd/resume-service

# Run all tests
go test ./...

# Run tests for a specific package
go test ./internal/resume/service/...

# Run a single test
go test ./internal/resume/service/... -run TestResumeService_Create

# Build the service
go build -o resume-service ./cmd/resume-service

# Generate protobuf code (after modifying proto/resume.proto)
protoc --go_out=. --go-grpc_out=. proto/resume.proto
```

## Architecture

This service follows a layered architecture with dependency injection:

```
cmd/resume-service/main.go
    └── wires up all dependencies and starts HTTP server

internal/
├── resume/
│   ├── handler/     # HTTP handlers (Gin) - receives requests, validates input
│   ├── service/     # Business logic - orchestrates operations, enforces rules
│   ├── repository/  # Data access - database operations (PostgreSQL via GORM)
│   └── model/       # Domain models and status transitions
└── shared/
    ├── storage/     # MinIO file storage client
    ├── events/      # Redis Streams event publisher
    ├── middleware/  # HTTP middleware (CORS, logging, recovery)
    ├── response/    # Standardized JSON response helpers
    ├── logger/      # Zap logger wrapper
    └── pb/          # Generated protobuf/gRPC code
```

## Key Patterns

**Layered Architecture Flow:**
- Handler → Service → Repository
- Each layer depends on interfaces, not concrete implementations
- Use constructor injection: `NewResumeHandler(svc)`, `NewResumeService(repo, storage, publisher)`

**Status State Machine:**
- Resumes have status: `pending` → `processing` → `parsed` or `failed` → `archived`
- Status transitions are validated in `model.Resume.CanTransitionTo()`
- See `internal/resume/model/resume.go` for valid transitions

**Event Publishing:**
- Events are published to Redis Streams via `EventPublisher`
- Event types: `created`, `updated`, `deleted`, `status_changed`, `parsed`
- Events are optional; service works without Redis

## Infrastructure Dependencies

- **PostgreSQL**: Primary database (via GORM with raw SQL for JSON handling)
- **MinIO**: S3-compatible object storage for resume files (PDF, DOC, DOCX)
- **Redis**: Event streaming (optional; service continues without it)

Default connection settings are configured in `main.go` and can be overridden via environment variables.

## Testing

- Uses `testify/mock` for mock implementations
- Mock types are defined in test files (e.g., `MockRepository`, `MockStorage`)
- Unit tests focus on service layer; handlers and repositories have their own tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lenoud) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
