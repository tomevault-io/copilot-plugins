## forum

> **Architecture**: Modular Monolith with Hexagonal Architecture (Ports & Adapters)

# Forum - AI Coding Agent Instructions

## Quick Reference

**Architecture**: Modular Monolith with Hexagonal Architecture (Ports & Adapters)  
**Language**: Go 1.24+ with minimal dependencies (SQLite driver, bcrypt, UUID only)  
**Status**: 75% Complete - Auth, Posts, Categories fully implemented with tests  
**Entry Point**: `cmd/forum/main.go` → `cmd/forum/wire/` for all DI wiring  
**Critical Files**: `docs/ARCHITECTURE.md`, `docs/IMPLEMENTATION_ROADMAP.md`, `docs/UNIFIED_DI_PATTERN.md`

---

## Architecture Pattern: The Hexagon

Each module follows **strict 4-directory structure**:

```
internal/modules/{module}/
├── domain/          # Pure business logic (stdlib only)
│   ├── {entity}.go  # Domain entities with Validate()
│   └── errors.go    # Domain-specific errors (errors.New())
├── ports/           # Interface definitions (contracts)
│   ├── service.go   # INPUT PORT - Use case interface
│   └── repository.go # OUTPUT PORT - Data access interface
├── application/     # Business logic orchestration
│   └── service.go   # Implements ports interfaces
└── adapters/        # Technical implementations (FLAT - no subdirs)
    ├── http_handler.go       # INPUT ADAPTER - HTTP handlers
    └── sqlite_repository.go  # OUTPUT ADAPTER - Database access
```

### Port/Adapter File Headers (MANDATORY)

Every file in `ports/` and `adapters/` MUST start with:
- `// INPUT PORT - Service Interface`
- `// OUTPUT PORT - Repository Interface`
- `// INPUT ADAPTER - HTTP Handler`
- `// OUTPUT ADAPTER - SQLite Repository`

**Example**: `internal/modules/auth/ports/service.go` starts with `// INPUT PORT - Service Interface`

### Dependency Rules (Enforced)

```
domain      → stdlib ONLY (no imports from project)
ports       → domain only
application → domain, ports
adapters    → domain, ports (never application!)
```

Cross-module: Import `ports.XService` interface ONLY. Never import `application/` or `adapters/` from other modules.

---

## Critical Workflow: Adding Features (Follow Exactly)

### 1. Domain Layer First
```go
// internal/modules/{module}/domain/{entity}.go
type Post struct {
    ID        string
    Title     string
    Content   string
    CreatedAt time.Time
}

func (p *Post) Validate() error {
    if p.Title == "" { return domain.ErrEmptyTitle }
    return nil
}

// internal/modules/{module}/domain/errors.go
var ErrEmptyTitle = errors.New("title cannot be empty")
```

### 2. Define Ports (Interfaces)
```go
// internal/modules/{module}/ports/service.go (INPUT PORT)
type PostService interface {
    CreatePost(ctx context.Context, userID, title, content string) (*domain.Post, error)
}

// internal/modules/{module}/ports/repository.go (OUTPUT PORT)
type PostRepository interface {
    Create(ctx context.Context, post *domain.Post) error
}
```

### 3. Implement Application Service
```go
// internal/modules/{module}/application/service.go
type Service struct {
    repo ports.PostRepository
}

func (s *Service) CreatePost(ctx context.Context, userID, title, content string) (*domain.Post, error) {
    post := &domain.Post{ID: generateID(), Title: title, Content: content}
    if err := post.Validate(); err != nil { return nil, err }
    return post, s.repo.Create(ctx, post)
}
```

### 4. Implement Adapters
```go
// internal/modules/{module}/adapters/sqlite_repository.go (OUTPUT ADAPTER)
type SQLiteRepository struct { db *sql.DB }
func (r *SQLiteRepository) Create(ctx context.Context, post *domain.Post) error {
    _, err := r.db.ExecContext(ctx, "INSERT INTO posts...", post.ID, post.Title)
    return err
}

// internal/modules/{module}/adapters/http_handler.go (INPUT ADAPTER)
type HTTPHandler struct { service ports.PostService }
func (h *HTTPHandler) CreatePostAPI(w http.ResponseWriter, r *http.Request) {
    // Parse request, call h.service.CreatePost(), write JSON response
}
```

### 5. Wire in `cmd/forum/wire/` (CRITICAL - Don't Skip!)
```go
// cmd/forum/wire/repositories.go
repos.Post = postAdapters.NewSQLitePostRepository(db)

// cmd/forum/wire/services.go (ServiceContainer already exists)
post: postApp.NewService(repos.Post, repos.Category),

// cmd/forum/wire/handlers.go
postHandler := postAdapters.NewHTTPHandler(services, templates)

// cmd/forum/wire/app.go (in initServer)
postHandler.RegisterRoutes(server.Router())
```

### 6. Database Migration (if schema changes)
```sql
-- migrations/NNN_{module}_{description}.sql
-- +migrate Up
CREATE TABLE posts (id TEXT PRIMARY KEY, title TEXT NOT NULL);
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- +migrate Down
DROP INDEX idx_posts_user_id;
DROP TABLE posts;
```

**Migrations run automatically on startup** via `cmd/forum/wire/app.go:initDatabase()`.

---

## Unified Dependency Injection Pattern (CRITICAL)

**Every handler uses identical ServiceContainer pattern**:

```go
// cmd/forum/wire/services.go - ONE concrete container with ALL services
type ServiceContainer struct {
    auth     authPorts.AuthService      // Private fields
    user     userPorts.UserService
    post     postPorts.PostService
}
func (sc *ServiceContainer) Auth() authPorts.AuthService { return sc.auth }
func (sc *ServiceContainer) User() userPorts.UserService { return sc.user }

// Each handler constructor: IDENTICAL signature across ALL handlers
func NewHTTPHandler(services ServiceContainer, templates *template.Template) *HTTPHandler

// Each handler declares minimal local interface (only what it needs)
type ServiceContainer interface {
    Post() postPorts.PostService
    Auth() authPorts.AuthService  // Only if needed for auth checks
}
```

**Benefits**: Consistent signatures, explicit dependencies, easy mocking, no circular imports.  
**Reference**: `docs/UNIFIED_DI_PATTERN.md`, `cmd/forum/wire/services.go`

---

## Handler Naming Convention

- **API handlers** (JSON): End with `API` → `RegisterAPI`, `LoginAPI`, `CreatePostAPI`
- **Page handlers** (HTML): End with `Page` → `LoginPage`, `RegisterPage`, `PostDetailPage`

Group in code: API handlers first, then Page handlers.

---

## Error Handling (Two Layers)

```go
// 1. Domain errors (module-level, simple)
// internal/modules/auth/domain/errors.go
var ErrSessionExpired = errors.New("session has expired")

// 2. Platform errors (structured, HTTP-aware)
// internal/platform/errors/errors.go
return errors.Wrap(err, errors.ErrCodeInternal, "failed to create session")
```

HTTP handlers map errors to status codes via `errors.HTTPStatus()`:
- 200 OK, 201 Created, 204 No Content
- 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found
- 409 Conflict (duplicate email/username)
- 500 Internal Server Error

Always return JSON: `{error: "message"}` for errors.

---

## Testing Strategy

**Run tests**: `make test` (runs `scripts/tests/run_all_tests.sh`)  
**Coverage**: `make test-coverage` (generates `coverage.html`)

```
tests/
├── unit/              # Business logic, isolated mocks
│   ├── auth_test.go          # Domain validation, service logic
│   └── template_*_test.go    # Template rendering tests
├── integration/       # Full request/response cycles
    ├── auth_test.go          # Register, login, session flows
    └── post_test.go          # CRUD operations with real DB
```

**TDD Workflow**:
1. Write failing test → `go test ./internal/modules/auth/... -run TestRegister`
2. Implement minimal code to pass
3. Refactor while keeping tests green
4. Commit

**Audit compliance**: ALL scenarios from `docs/requirements/audit.md` MUST have integration tests. DO NOT modify audit.md.

---

## Build & Run

**Local development**:
```bash
make go              # Runs with `go run cmd/forum/main.go`
make build           # Builds binary to bin/forum
make run             # Builds then runs binary
```

**Docker**:
```bash
docker-compose up    # Starts forum + database
make docker-build    # Builds Docker image
```

**Important**: SQLite requires `CGO_ENABLED=1` (set in Makefile).

---

## Configuration & Logging

**Config**: `internal/platform/config/config.go` - loads from env vars with defaults  
**Logger**: `internal/platform/logger/` - structured logging (Debug, Info, Warn, Error)

```go
lgr.Info("Starting service", logger.String("module", "auth"))
lgr.Error("Failed operation", logger.Error(err), logger.String("user_id", uid))
```

**Middleware order** (in `cmd/forum/wire/app.go`):
1. Recovery (panic handling) - TODO: Not yet implemented
2. Logger (request logging) - TODO: Not yet implemented
3. CORS
4. RateLimit


---

## Key Files Reference

- **Entry point**: `cmd/forum/main.go` - Minimal lifecycle (config → wire → start → shutdown)
- **DI wiring**: `cmd/forum/wire/{app,repositories,services,handlers}.go` - All dependency injection
- **Architecture**: `docs/ARCHITECTURE.md` - Design rationale, dependency rules
- **Roadmap**: `docs/IMPLEMENTATION_ROADMAP.md` - Progress tracking (75% complete)
- **DI pattern**: `docs/UNIFIED_DI_PATTERN.md` - ServiceContainer details
- **Auth reference**: `internal/modules/auth/` - Fully implemented example (ports → app → adapters)
- **Audit spec**: `docs/requirements/audit.md` - Authoritative test scenarios (DO NOT modify)
- **Logger plan**: `docs/log/UNIFIED_LOGGER_IMPROVEMENT_PLAN.md` - Middleware implementation TODO

---

## Current Implementation Status (75% Complete)

**✅ Fully Implemented**:
- Auth module (register, login, sessions, validation, middleware)
- Post module (CRUD with categories, validation)
- Category module (CRUD operations)
- Platform layer (config, database, logger, httpserver, errors, validator)
- Database migrations (auto-apply on startup)
- Unit + integration tests for auth and posts

**⚠️ Placeholder/TODO** (look for `// TODO:` comments):
- Comment module (scaffolded, not implemented)
- Reaction module (scaffolded, not implemented)
- User module HTTP handlers (domain + repo done, handlers TODO)
- Moderation module (optional, scaffolded)
- Notification module (optional, scaffolded)
- Recovery & Logger middleware (`internal/platform/httpserver/middleware.go`)

**When implementing**: Replace `// TODO:` placeholders with actual logic. Follow the hexagonal pattern exactly as shown in auth/post modules.

---

## Development Workflow

**Adding features**:
1. Check `docs/IMPLEMENTATION_ROADMAP.md` for current priorities
2. Follow the 6-step workflow (Domain → Ports → Application → Adapters → Wire → Migration)
3. Use `internal/modules/auth/` as reference implementation
4. Update roadmap checkboxes as you complete tasks

**Common commands**:
```bash
make go              # Run locally (go run cmd/forum/main.go)
make test            # Full test suite
make test-coverage   # Generate coverage.html
make fmt             # Format code
make vet             # Static analysis
```

**Testing workflow**:
```bash
# Run specific module tests
go test ./internal/modules/auth/... -v
# Run with race detector
go test -race ./...
# Integration tests only
go test ./tests/integration/... -v
```

**Git commits**:
- Format: `[module] Brief description`
- Example: `[auth] Implement session validation`, `[post] Add category filtering`
- Commit after each complete feature/checklist item

---

## Common Pitfalls

❌ **Don't do**:
- Add subdirectories to `adapters/` (keep flat: http_handler.go, sqlite_repository.go)
- Skip port/adapter file header comments (`// INPUT PORT - Service Interface`, etc.)
- Import `application/` from adapters (adapters → ports only)
- Use external frameworks unnecessarily (prefer stdlib)
- Modify `docs/requirements/{audit,requirements,morefeats}.md` (authoritative specs)
- Hard-code config values (use `config.Config` structs)

✅ **Do**:
- Follow exact 4-directory structure for every module
- Declare handler-specific ServiceContainer interfaces
- Use domain errors in modules, platform errors for HTTP
- Write tests before/during implementation (TDD)
- Check existing modules (auth, post) for patterns
- Mark optional features with `[OPTIONAL FEATURE]` comments

---

## Quick Patterns

**Domain validation**:
```go
func (p *Post) Validate() error {
    if p.Title == "" { return domain.ErrEmptyTitle }
    if len(p.Title) > 300 { return domain.ErrTitleTooLong }
    if len(p.Categories) == 0 { return domain.ErrNoCategoriesSelected }
    return nil
}
```

**Service implementation**:
```go
func (s *Service) CreatePost(ctx context.Context, userID, title, content string, categories []string) (*domain.Post, error) {
    post := &domain.Post{ID: generateID(), UserID: userID, Title: title, Content: content, Categories: categories}
    if err := post.Validate(); err != nil { return nil, err }
    return post, s.postRepo.Create(ctx, post)
}
```

**HTTP handler pattern**:
```go
func (h *HTTPHandler) CreatePostAPI(w http.ResponseWriter, r *http.Request) {
    // 1. Parse & validate input
    // 2. Extract user from session (if auth required)
    // 3. Call service method
    // 4. Handle errors with appropriate status codes
    // 5. Return JSON response
}
```

**Repository pattern**:
```go
func (r *SQLiteRepository) Create(ctx context.Context, post *domain.Post) error {
    tx, _ := r.db.BeginTx(ctx, nil)
    defer tx.Rollback()
    _, err := tx.ExecContext(ctx, "INSERT INTO posts (id, title, content, user_id) VALUES (?, ?, ?, ?)", ...)
    return tx.Commit()
}
```

---

## Project Structure Summary

```
forum/
├── cmd/forum/
│   ├── main.go                    # Entry point (config → wire → run)
│   └── wire/                      # Dependency injection (repos → services → handlers)
├── internal/
│   ├── modules/                   # Business modules (hexagonal structure)
│   │   ├── auth/                  # ✅ Auth (fully implemented)
│   │   ├── post/                  # ✅ Posts (fully implemented)
│   │   ├── comment/               # ⚠️ TODO: Implement
│   │   └── reaction/              # ⚠️ TODO: Implement
│   └── platform/                  # ✅ Technical infrastructure
│       ├── config/                # Env-based config
│       ├── database/              # SQLite connection + migrator
│       ├── errors/                # Structured error types
│       ├── httpserver/            # HTTP server + middleware
│       ├── logger/                # Structured logging
│       └── validator/             # Input validation
├── migrations/                    # SQL migrations (auto-applied)
├── templates/                     # HTML templates (Go html/template)
├── static/                        # CSS, JS, uploads
├── tests/
│   ├── unit/                      # Business logic tests
│   └── integration/               # Full HTTP cycle tests
└── docs/
    ├── ARCHITECTURE.md            # Design decisions
    ├── IMPLEMENTATION_ROADMAP.md  # Progress tracking
    └── requirements/              # Specs (DO NOT modify)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ertval) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
