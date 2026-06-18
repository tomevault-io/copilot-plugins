## aura-back

> - **Language**: Go 1.26.1 | **Framework**: Gin (github.com/gin-gonic/gin)

# AGENTS.md - Agent Coding Guidelines for aura-back

## Project Overview

- **Language**: Go 1.26.1 | **Framework**: Gin (github.com/gin-gonic/gin)
- **Database**: PostgreSQL with `lib/pq` driver | **Migrations**: golang-migrate/v4
- **Architecture**: Multi-tenant SaaS with schema-per-tenant pattern
- **Testing**: stretchr/testify + DATA-DOG/go-sqlmock

## Essential Commands

```bash
# Run the application
go run ./cmd/api/main.go

# Build
go build ./...

# Run all tests
go test ./...

# Run single test (most important for agents)
go test -v -run TestService_Create_ValidSlugFormat ./modules/enterprise/...
go test -v -run TestListRolesByMinLevel ./modules/users/...

# Run tests with race detection
go test -race ./...

# Run tests with coverage
go test -cover ./...
go test -coverprofile=coverage.out ./... && go tool cover -html=coverage.out

# Formatting & vetting (run these before committing)
go fmt ./...
go vet ./...
go mod tidy
```

## Project Structure

```
aura-back/
├── cmd/api/main.go              # Entry point & dependency wiring
├── cmd/server/server.go         # Router, middleware, module registration
├── internal/db/db.go            # Database connection pool
├── shared/                      # Cross-cutting: errors, events, response, domain/vo
├── tenant/                      # Multi-tenant: Manager, Auth, Middleware, migrations/
│   ├── migrations/public/       # Public schema migrations
│   └── migrations/tenant/       # Tenant schema migrations
├── modules/                     # Feature modules (self-contained)
│   └── enterprise/              # Each module: domain.go, service.go, repository.go, handler.go, routes.go
└── infrastructure/messaging/    # Event bus implementations
```

## Module Pattern

Modules can be simple standalone directories or grouped as sub-modules within a parent directory.
Each feature module (e.g., `modules/<name>/` or `modules/<group>/<name>/`) follows this structure:
- `domain.go` - Entity, Repository interface, Service interface, events
- `service.go` - Business logic (unexported struct, constructor returns interface)
- `repository.go` - PostgreSQL implementation with `querier` interface for DB/Tx support
- `handler.go` - Gin HTTP handlers with request/response types
- `routes.go` - `Register(public, protected gin.IRouter, h *Handler)` function

## Code Style

### Imports (3 groups, blank line between)
```go
import (
    "context"
    "database/sql"
    "fmt"

    "github.com/gin-gonic/gin"
    "github.com/lib/pq"

    "github.com/cloud-tech-develop/aura-back/shared/errors"
    "github.com/cloud-tech-develop/aura-back/tenant"
)
```

### Naming
- **Exported**: PascalCase (`Service`, `NewHandler`, `TenantKey`)
- **Unexported**: camelCase (`repository`, `eventBus`, `validSlug`)
- **Packages**: lowercase, no underscores (`shared`, `enterprise`, `vo`)
- **Interfaces**: -er suffix for single methods (`Migrator`, `Logger`)
- **Constants**: Group in `const` blocks; event names as `"module.action"` strings

### Error Handling
```go
// Wrap with context using %w
if err != nil {
    return fmt.Errorf("crear tenant: %w", err)
}

// Use sql.ErrNoRows for "not found" - callers check with errors.Is()
if err == sql.ErrNoRows {
    return nil, sql.ErrNoRows
}

// Sentinel errors in shared/errors/ for common cases
// Service returns domain errors; handler maps to HTTP status
if errors.Is(err, ErrPlanLimitReached) {
    response.Forbidden(c, err.Error())
    return
}
```

### Database Operations
- Always use `QueryRowContext`/`ExecContext` with context parameter
- Use parameterized queries (`$1, $2`) - never string interpolation
- Close rows: `defer rows.Close()` immediately after Query
- Use transactions for multi-step operations: `tx, err := db.BeginTx(ctx, nil)`
- Support both DB and Tx via `querier` interface pattern

### HTTP Handlers
```go
// Read tenant from context (middleware sets it)
slug, ok := tenant.SlugFromContext(c)
enterpriseID := c.GetInt64("enterprise_id")

// Use shared/response helpers for consistent responses
response.OK(c, data)
response.Created(c, data)
response.BadRequest(c, msg)
response.NotFound(c, msg)
response.Conflict(c, msg)

// Bind and validate JSON
var req createRequest
if err := c.ShouldBindJSON(&req); err != nil {
    response.BadRequest(c, err.Error())
    return
}
```

### Context Usage
- Always pass `ctx context.Context` as first parameter
- Use `c.Request.Context()` in handlers (never `context.Background()`)
- Never store contexts in structs

## Multi-Tenant Pattern

- Tenant identified by slug in JWT claims or subdomain
- Each tenant gets PostgreSQL schema named after slug (`empresa_uno`)
- Tenant middleware validates slug and sets context
- Public tables (users, enterprises, roles) in `public` schema
- Tenant tables (third_parties, products) in tenant schema
- Email uniqueness enforced across all tenants at service level

## Testing Conventions

```go
// Test naming: Test<Component>_<Method>_<Scenario>
func TestService_Create_DuplicateSlug(t *testing.T) { ... }
func TestListRolesByMinLevel_SuperAdmin(t *testing.T) { ... }

// Use table-driven tests for multiple cases
tests := []struct {
    name    string
    slug    string
    wantErr bool
}{
    {"valid lowercase", "empresa_uno", false},
    {"invalid chars", "empresa@test", true},
}
for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) { ... })
}

// Mock repositories with testify/mock
type MockRepository struct { mock.Mock }
func (m *MockRepository) GetBySlug(ctx context.Context, slug string) (*Enterprise, error) {
    args := m.Called(ctx, slug)
    if args.Get(0) == nil { return nil, args.Error(1) }
    return args.Get(0).(*Enterprise), args.Error(1)
}

// Use go-sqlmock for repository tests
db, mock, err := sqlmock.New()
```

## Adding a New Module

### Adding a Simple Module
1. Create `modules/<name>/` with domain.go, service.go, repository.go, handler.go, routes.go
2. Define entity, repository interface, service interface in domain.go
3. Implement service (unexported struct, `NewService` returns interface)
4. Implement repository with `querier` interface for transaction support
5. Create handler with `ShouldBindJSON` for request validation
6. Add routes in `Register(public, protected gin.IRouter, h *Handler)`
7. Create migration SQL files in `tenant/migrations/tenant/`
8. Register handler in `cmd/api/main.go` and `cmd/server/server.go`

### Adding a Sub-Module (Grouped)
1. Create the sub-directory `modules/<group>/<name>/`
2. Follow the same file pattern (`domain.go`, `service.go`, etc.) with `package <name>`
3. Register the handler and service **independently** alongside other sub-modules in `cmd/api/main.go`
4. Add the `<name>.Register(...)` call in `RegisterModules` inside `cmd/server/server.go`

## Security

- Never hardcode credentials; use `.env` + `os.Getenv()`
- Passwords hashed with bcrypt (`tenant.HashPassword`)
- JWT includes user_id, enterprise_id, slug, roles, role_level, ip
- IP validation in JWT claims (prevents token theft)
- Parameterized queries prevent SQL injection
- Input validation at handler level, business validation at service level

## Dependencies

Key packages: `gin-gonic/gin`, `lib/pq`, `golang-migrate/v4`, `golang-jwt/jwt/v5`, `stretchr/testify`, `DATA-DOG/go-sqlmock`, `joho/godotenv`

## Pagination Pattern

All page endpoints must use `shared/domain.PageResult` for consistent response format.

### Structures (in shared/domain/pagination.go)

```go
// PageParams holds pagination request parameters
type PageParams struct {
    First  int64  `json:"first"` // offset (starting from 1)
    Rows   int64  `json:"rows"`  // limit
    Search string `json:"search"`
}

// PageResult holds a paginated result with items and pagination metadata
type PageResult struct {
    Items      interface{} `json:"items"`
    Total      int64       `json:"total"`
    Page       int64       `json:"page"`
    Limit      int64       `json:"limit"`
    TotalPages int64       `json:"totalPages"`
}
```

### Response Format

All page endpoints must return:

```json
{
  "data": {
    "items": [...],
    "total": 15,
    "page": 1,
    "limit": 10,
    "totalPages": 2
  },
  "message": "Operacion exitosa",
  "success": true
}
```

### Implementation Requirements

1. **Repository**: Execute a COUNT query first to get total elements, then the SELECT with LIMIT/OFFSET
2. **Filters**: Always filter by `deleted_at IS NULL` and:
   - For categories/products: `active = true` or `status = 'ACTIVE'`
   - For brands: no additional filter (just active by default)
3. **Module interfaces**: Use `domain.PageResult` as return type in Repository and Service interfaces
4. **Validation**: In service, validate that `first >= 1` and `rows >= 1`, defaulting to 1 and 10 respectively

## Cross-Database Types (vo package)

When working with both PostgreSQL and SQLite (offline mode), use the types in `shared/domain/vo/`:

### vo.DateTime

For fields that store timestamps, use `vo.DateTime` instead of `time.Time`:

```go
import "github.com/cloud-tech-develop/aura-back/shared/domain/vo"

type Product struct {
    CreatedAt vo.DateTime  `json:"created_at"`
    UpdatedAt *vo.DateTime `json:"updated_at"`
}
```

`vo.DateTime` handles:
- PostgreSQL timestamps (`time.Time`)
- SQLite string timestamps with various formats
- Monotonic clock suffixes (`m=+8.005836901`)
- Timezone duplication issues

### Nullable String Fields

For fields that can be NULL (from LEFT JOINs), use `*string` instead of `string`:

```go
type Product struct {
    BrandName     *string `json:"brand_name"`
    CategoryName *string `json:"category_name"`
    UnitName     *string `json:"unit_name"`
}
```

## Tenant Context Functions

In `tenant/middleware.go`, use these functions to get authenticated user data:

```go
// Get tenant slug from context or JWT token
slug, ok := tenant.SlugFromContext(c)

// Get enterprise ID from context or JWT token
enterpriseID, ok := tenant.EnterpriseIDFromContext(c)

// Get all token claims
claims, ok := tenant.ClaimsFromContext(c)
// claims.UserID, claims.EnterpriseID, claims.Slug, claims.Email, claims.Roles, etc.
```

---
> Source: [cloud-tech-develop/aura-back](https://github.com/cloud-tech-develop/aura-back) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
