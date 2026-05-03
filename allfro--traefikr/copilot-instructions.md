## traefikr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Traefikr is a Traefik configuration management API that provides a REST API for managing Traefik v3.6 resources (routers, services, middlewares, etc.) with comprehensive JSON schema validation. The backend is written in Go and uses SQLite for persistence.

## Build and Development Commands

### Docker Development
```bash
# Build and start all services (Traefik + Backend)
docker-compose up --build

# Rebuild backend only
docker-compose build backend

# Stop and remove containers
docker-compose down

# Remove backend data volume
docker volume rm traefikr_backend-data
```

### Backend Development
```bash
cd backend

# Install dependencies
go mod download

# Build binary
go build -o traefikr .

# Build static binary for production (used in Dockerfile)
CGO_ENABLED=1 GOOS=linux go build \
    -a \
    -ldflags '-linkmode external -extldflags "-static" -s -w' \
    -o main .

# Run locally
TRAEFIKR_DB_PATH=./traefikr.db TRAEFIKR_PORT=8080 TRAEFIK_API_URL=http://localhost:8080 ./traefikr
```

### Testing
```bash
# Test authentication flows
./tests/test_auth.sh

# Create test resources
./tests/create_fixed_resources.sh

# Create all middleware types
./tests/create_middlewares.sh

# Delete all resources
./tests/delete_all_resources.sh
```

## Architecture

### Dual Authentication System

Traefikr implements **two separate authentication mechanisms** - this is critical to understand:

1. **JWT Authentication** (User/Admin access)
   - Used for all CRUD operations on resources
   - Login at `/api/auth/login` with username/password returns JWT token
   - Token must be passed as `Authorization: Bearer <token>` header
   - Tokens expire after 24 hours
   - Implemented in `middleware/jwt.go`

2. **API Key Authentication** (Traefik polling)
   - **ONLY** used for `/api/config` endpoint (Traefik HTTP provider polling)
   - API keys are standalone, not linked to users
   - Created via `/api/http/provider` endpoint (requires JWT)
   - Passed as `x-traefikr-key` header
   - Implements conditional authentication: public if no API keys exist (bootstrap mode), requires auth once keys are created
   - Implemented in `middleware/auth.go`

**IMPORTANT**: These two authentication systems must remain separate. Do not mix them or allow x-traefikr-key to be used for CRUD operations.

### Route Structure

```
Public Endpoints:
- GET  /health                              # Health check
- POST /api/auth/login                      # Get JWT token
- GET  /api/{protocol}/{type}/schema.json   # Get validation schema

Conditional Auth (API Key for Traefik):
- GET  /api/config                          # Traefik HTTP provider endpoint

Protected Endpoints (JWT Required):
- GET    /api/{protocol}/{type}                    # List resources (supports ?traefik=true|false)
- GET    /api/{protocol}/{type}/{nameProvider}     # Get resource
- POST   /api/{protocol}/{type}                    # Create resource
- PUT    /api/{protocol}/{type}/{nameProvider}     # Update resource
- DELETE /api/{protocol}/{type}/{nameProvider}     # Delete resource
- GET    /api/entrypoints                          # List entrypoints
- GET    /api/entrypoints/{name}                   # Get entrypoint
- GET    /api/http/provider                        # List API keys
- POST   /api/http/provider                        # Create API key
- DELETE /api/http/provider/{id}                   # Delete API key
```

### Query Parameters

The List Resources endpoint (`GET /api/{protocol}/{type}`) supports an optional `traefik` query parameter:

- **traefik=false** (default): Returns only resources from the local database
- **traefik=true**: Returns resources from both database and Traefik API (merged)

Examples:
```bash
# Get only database resources (default)
curl -H "Authorization: Bearer $JWT" http://localhost:8000/api/http/routers

# Explicitly request only database resources
curl -H "Authorization: Bearer $JWT" http://localhost:8000/api/http/routers?traefik=false

# Get merged resources from database and Traefik
curl -H "Authorization: Bearer $JWT" http://localhost:8000/api/http/routers?traefik=true
```

### Protocol and Resource Types

Supported protocols: `http`, `tcp`, `udp`

Resource types by protocol:
- **http**: routers, services, middlewares, serversTransport, tls
- **tcp**: routers, services, middlewares, serversTransport, tls
- **udp**: routers, services, middlewares

### Resource Naming Convention

Resources are identified by `name@provider` format (e.g., `my-router@http`). When creating resources:
- Request body contains `name` field without `@provider` suffix
- Response includes full `name@provider` identifier
- Database stores name and provider separately

### Data Flow

1. **Resource Creation/Update**:
   - User authenticates with JWT
   - Request validated against JSON schema (in `schemas/`)
   - Resource stored in SQLite database
   - Traefik polls `/api/config` to get updated configuration

2. **Resource Listing**:
   - Fetches resources from both Traefik API (live) and local database
   - Merges results, prioritizing database for enabled/disabled state
   - Traefik client in `utils/traefik_client.go` handles API communication

3. **Configuration Export**:
   - `/api/config` reads all enabled resources from database
   - Transforms into Traefik HTTP provider format
   - Groups by protocol (http/tcp/udp) and type (routers/services/etc.)
   - Returns configuration in format: `{protocol: {type: {name: config}}}`

### Database Models

- **User**: Admin users with bcrypt-hashed passwords
- **APIKey**: Standalone API keys for Traefik polling (not linked to users)
- **TraefikConfig**: Stores resource configurations with composite primary key (name, provider, protocol, type)
  - This allows the same resource name to exist across different protocols and types
  - Example: `test-router` can exist as both HTTP router and TCP router simultaneously

### JSON Schema Validation

All resource configurations are validated against embedded JSON schemas before persistence:
- Schemas located in `backend/schemas/` directory
- One schema file per protocol/type combination
- Validation enforced on create and update operations
- Schemas embedded at compile time using `//go:embed`

### Container Architecture

The backend runs in a **FROM scratch** container with:
- Fully static, stripped Go binary (no external dependencies)
- No shell or package manager (security hardened)
- SQLite compiled statically using musl
- Binary size: ~36MB, runtime memory: ~8-12 MiB
- Data persisted to `/data/traefikr.db` inside container
- Volume mounted to host at `/var/lib/docker/volumes/traefikr_backend-data/_data/`

## Environment Variables

Backend (`docker-compose.yml` or local execution):
```bash
TRAEFIKR_DB_PATH=/data/traefikr.db                    # Database location
TRAEFIKR_PORT=8080                                     # Server port
TRAEFIK_API_URL=http://traefik:8080                   # Traefik API endpoint
TRAEFIKR_JWT_SECRET=<random>                          # JWT signing key (auto-generated 256-bit key if not set)
TRAEFIK_CONFIG_PATH=/traefik/dynamic                  # Path to write serverTransport TOML files (optional)

# Traefik API Authentication (optional)
TRAEFIK_BASIC_AUTH_USERNAME=<username>                # Basic auth username for Traefik API
TRAEFIK_BASIC_AUTH_PASSWORD=<password>                # Basic auth password for Traefik API
TRAEFIK_API_KEY_HEADER=<header-name>                  # API key header name (e.g., X-API-Key)
TRAEFIK_API_KEY_SECRET=<api-key>                      # API key value
```

### Traefik API Authentication

If your Traefik API is protected with authentication, configure the appropriate environment variables:

- **Basic Authentication**: Set `TRAEFIK_BASIC_AUTH_USERNAME` and `TRAEFIK_BASIC_AUTH_PASSWORD`. When the username is defined, all requests to the Traefik API will include Basic authentication headers.

- **API Key Authentication**: Set `TRAEFIK_API_KEY_HEADER` (the header name) and `TRAEFIK_API_KEY_SECRET` (the key value). When the header is defined, all requests to the Traefik API will include this custom header.

- **No Authentication**: If neither is configured, requests to the Traefik API will be sent without authentication headers (default behavior).

## Initial Setup

On first run, the backend:
1. Creates SQLite database and runs migrations
2. Generates admin user with random password
3. Prints credentials to stdout (save these!)
4. Starts HTTP server on configured port

Example output:
```
==================================================
Initial Admin Credentials
Username: admin
Password: jSmiVnQZ5LL0m-8x
==================================================
Please save these credentials! The password will not be shown again.
```

## Code Organization

```
backend/
├── main.go                      # Entry point, route setup
├── dal/                         # Data Access Layer (Repository pattern)
│   ├── api_key_repository.go   # API key database operations
│   ├── bootstrap.go             # Startup bootstrap operations
│   ├── traefik_config_repository.go # Traefik config database operations
│   └── user_repository.go      # User database operations
├── handlers/                    # HTTP request handlers
│   ├── auth.go                 # Login endpoint
│   ├── config.go               # Traefik config export
│   ├── entrypoints.go          # Entrypoint read-only operations
│   ├── provider.go             # API key management
│   └── resources.go            # CRUD operations for resources
├── middleware/                  # Authentication middleware
│   ├── auth.go                 # API key authentication
│   └── jwt.go                  # JWT authentication
├── models/                      # Database models and initialization
│   ├── api_key.go              # API key model
│   ├── database.go             # DB initialization
│   ├── traefik_config.go       # Resource configuration model
│   └── user.go                 # User model
├── schemas/                     # Embedded JSON schemas for validation
│   ├── validator.go            # Schema validation logic
│   └── *.json                  # Schema files (embedded at compile time)
├── utils/                       # Utilities
│   ├── toml_writer.go          # ServerTransport TOML file writer
│   └── traefik_client.go       # Traefik API client
├── Dockerfile                   # FROM scratch static binary build
└── openapi.json                 # OpenAPI 3.1 specification
```

## Important Implementation Notes

1. **Backend is FROZEN**: The backend codebase is production-ready and should not be modified without explicit approval.

2. **Authentication Separation**: Never mix JWT and API key authentication. Each has a specific purpose and scope.

3. **Schema Validation**: All resource configs must pass JSON schema validation. Schemas are sourced from Traefik v3.6 documentation.

4. **Resource Naming**: Always use `name@provider` format in URLs and responses, but accept just `name` in request bodies.

5. **Conditional Auth**: The `/api/config` endpoint uses `ConditionalAuthMiddleware` which allows unauthenticated access when no API keys exist (bootstrap mode).

6. **Static Binary**: The Dockerfile produces a fully static binary. Do not add dependencies that require dynamic linking.

7. **Traefik Integration**: The backend acts as a Traefik HTTP provider. Traefik polls `/api/config` every 5 seconds.

---
> Source: [allfro/traefikr](https://github.com/allfro/traefikr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
