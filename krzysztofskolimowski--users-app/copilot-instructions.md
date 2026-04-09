## users-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go microservice for user management following hexagonal architecture and Domain-Driven Design (DDD) patterns. The service implements CQRS (Command Query Responsibility Segregation) with event-driven architecture for notifications.

## Architecture

**Hexagonal Architecture Layers:**
- `domain/` - Core business logic, entities, and interfaces
- `service/` - Application layer coordinating domain operations (split into commands and queries)
- `adapters/` - Infrastructure implementations (database, Redis, logging)
- `ports/` - External interfaces (HTTP REST and gRPC servers)
- `gen/` - Generated code from OpenAPI and Protocol Buffers

**Key Components:**
- **Domain Events**: User operations (add/modify/delete) publish events via Redis
- **CQRS Pattern**: Separate command (write) and query (read) services
- **Dual Interfaces**: Both REST HTTP and gRPC APIs available
- **Event Logging**: Events are logged to files and published to Redis channel

## Development Commands

**Environment Setup:**
- Uses `.env` file for configuration (see `.env.example`)
- Requires Docker, docker-compose, make, protoc, oapi-codegen, and gotestsum

**Build and Development:**
```bash
# Start application with Docker Compose
make up

# Stop application
make down

# Generate code from OpenAPI and Proto definitions
make generate

# Run tests (requires running PostgreSQL)
make test                    # Uses gotestsum
cd internal && go test ./... # Alternative without gotestsum

# Health check
curl -v http://localhost:8080/health
```

**Code Generation Dependencies:**
- `oapi-codegen` for OpenAPI/Swagger → Go HTTP server code
- `protoc` with Go plugins for Protocol Buffers → gRPC code

**gRPC Testing:**
```bash
make evans  # Interactive gRPC client
```

## Environment Configuration

**Service Control:**
- `RUN_HTTP=true/false` - Enable/disable HTTP server (default: true)
- `RUN_GRPC=true/false` - Enable/disable gRPC server (default: true)

**Database & Infrastructure:**
- PostgreSQL for data persistence
- Redis for event publishing
- File-based event logging

## Testing Strategy

The project implements a multi-layer testing approach:
- **Unit tests**: Domain logic testing
- **Integration tests**: Repository layer tests against real PostgreSQL
- Integration tests require running database (started by `docker compose up`)
- Set `DB_TEST_HOST` environment variable for test database connection

## Code Patterns

**Domain-Driven Design:**
- Rich domain models in `domain/user.go`
- Repository pattern interface in domain, implemented in adapters
- Domain events for cross-service communication

**Event System:**
- Commands implement `domain.Command` interface with `EncodeEvent()` method
- Events are wrapped with logging and publishing decorators
- Event flow: Command → Event → Redis publication + File logging

**CQRS Implementation:**
- `service.UsersCommandService` for write operations
- `service.UsersQueryService` for read operations
- Commands are separate from queries with different optimization paths

## Generated Code

Do not manually edit files in `internal/gen/`:
- `gen/api/` - Generated from `api/users.yml` OpenAPI specification
- `gen/grpc/` - Generated from `api/users.proto` Protocol Buffers definition

Run `make generate` after modifying API definitions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/krzysztofSkolimowski)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/krzysztofSkolimowski)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
