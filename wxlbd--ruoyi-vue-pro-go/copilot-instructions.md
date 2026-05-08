## ruoyi-vue-pro-go

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Go implementation of 芋道商城 (Yudao Mall), an e-commerce backend API that aligns with the Java-based ruoyi-vue-pro project. It provides RESTful APIs for an online shopping mall system using Go + Gin + GORM.

## Common Development Commands

### Build and Run
```bash
# Build the application
make build

# Run directly
make run

# Development with hot reload (requires Air)
make dev

# Download/update dependencies
make deps

# Regenerate Wire dependency injection
make wire

# Generate GORM DAO code
make gen

# Clean build artifacts
make clean
```

### Testing
**Note**: Currently no test files exist in the project. When implementing tests:
- Unit tests: `go test ./internal/...`
- Integration tests: `go test -tags=integration ./...`

## Architecture Overview

### Clean Architecture Pattern
The project follows clean architecture principles with clear separation of concerns:

```
HTTP Request → Handler → Service → Repository → Database
                    ↓
              Response DTO
```

### Directory Structure
- `cmd/server/` - Application entry point with Wire dependency injection
- `internal/api/` - HTTP layer (handlers, routers, request/response DTOs)
- `internal/service/` - Business logic layer
- `internal/repository/` - Data access layer using GORM Gen
- `internal/model/` - Data models/entities
- `internal/pkg/` - Internal packages (core utilities, middleware)
- `pkg/` - Public packages (config, logger)

### API Structure
- **Admin APIs**: `/api/admin/*` - Backend management
- **App APIs**: `/api/app/*` - Customer-facing features
- **Common APIs**: `/api/*` - Shared functionality

### Key Components

#### Authentication & Authorization
- JWT-based authentication with local validation
- User types: Member (0) and Admin (1)
- Token sources: Header, Query, Form parameters
- Tenant isolation support

#### Response Format
All APIs return standardized responses:
```json
{
    "code": 0,      // 0=success, others=error codes
    "msg": "",      // Message
    "data": {}      // Response data
}
```

#### Error Codes
- 0: Success
- 400: Parameter error
- 401: Unauthorized
- 403: Forbidden
- 404: Not found
- 409: Conflict
- 500: Server error

### Technology Stack
- **Framework**: Gin 1.11.0
- **ORM**: GORM with MySQL driver
- **Database**: MySQL 8.0+
- **Cache**: Redis 6.0+
- **DI**: Google Wire 0.7.0
- **Validation**: go-playground/validator
- **Logging**: Uber Zap with rotation
- **Config**: Viper

### Development Patterns

#### Adding New Features
1. Define models in `internal/model/`
2. Generate DAO code: `make gen`
3. Implement repository in `internal/repository/`
4. Create service in `internal/service/`
5. Add handlers in `internal/api/handler/`
6. Register routes in `internal/api/router/`
7. Wire dependency in `cmd/server/wire.go`
8. Regenerate wire: `make wire`

#### Database Operations
Use GORM Gen for type-safe queries:
```go
query := repo.NewQuery(db)
user, err := query.User.Where(query.User.ID.Eq(userID)).First()
```

#### Parameter Validation
Use struct tags for validation:
```go
type Request struct {
    Name string `json:"name" binding:"required,min=2,max=50"`
}
```

#### Error Handling
Use the standardized error system:
```go
core.WriteError(c, core.ParamErrCode, "参数错误")
core.WriteSuccess(c, data)
```

### Important Notes

1. **Java Alignment**: This Go implementation maintains 97% alignment with the Java version. Always verify API structures match the Java implementation.

2. **Wire Dependency Injection**: After modifying `cmd/server/wire.go`, always run `make wire` to regenerate the dependency injection code.

3. **GORM Code Generation**: After modifying models, run `make gen` to regenerate type-safe DAO code.

4. **No Testing Framework**: The project currently lacks tests. Consider this when making changes.

5. **Configuration**: Uses Viper with YAML config files and environment variable support.

6. **Logging**: Structured logging with Zap, automatically rotates logs.

7. **Middleware Stack**: ErrorHandler → Recovery → APIAccessLog → Auth (for protected routes)

### Common Tasks

#### Adding a New API Endpoint
1. Create handler in appropriate directory (admin/app)
2. Define request/response structs in `internal/api/req/` or `resp/`
3. Add route registration in `internal/api/router/`
4. Implement business logic in service layer
5. Wire dependencies in `cmd/server/wire.go`

#### Database Schema Changes
1. Update models in `internal/model/`
2. Run `make gen` to regenerate DAO code
3. Update repository methods if needed
4. Ensure service layer handles changes

#### Configuration Changes
1. Update `config/config.local.yaml`
2. Modify struct in `pkg/config/` if adding new config
3. Access via `config.C.YourConfig`

---
> Source: [wxlbd/ruoyi-vue-pro-go](https://github.com/wxlbd/ruoyi-vue-pro-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
