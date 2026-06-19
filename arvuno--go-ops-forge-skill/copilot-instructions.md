## go-ops-forge-skill

> **Go Version:** 1.22+

# GoOps Forge Skill

**Version:** 1.0
**Go Version:** 1.22+
**Target Environment:** Linux (production deployments)

---

## 1. Skill Identity

### Name
GoOps Forge Skill

### Purpose
Teaches AI coding agents how to design, scaffold, review, and improve production-grade Go projects for Linux environments. Covers APIs, CLIs, daemons, workers, schedulers, webhooks, gRPC services, and AI agent runtimes.

### Target Projects
- REST/gRPC APIs
- CLI tools
- Worker daemons and queue consumers
- Scheduled job services
- Webhook receivers
- Linux system agents
- MCP backends and agent runtimes
- DevOps and internal tooling
- Backend-for-frontend services

### Non-Goals
- Toy scripts or learning exercises
- Windows-specific GUI applications
- Mobile applications
- Projects explicitly marked "not for production"
- Experiments that will be discarded

---

## 2. When to Activate This Skill

Activate when the user asks for anything matching these patterns:

| User Request | Skill Action |
|-------------|-------------|
| "build a Go CLI" | Use `templates/cli/`; follow `recipes/create-new-cli.md` |
| "build a Go API server" | Use `templates/rest-api/`; follow `recipes/create-new-api-service.md` |
| "build a Go worker/consumer" | Use `templates/worker-daemon/`; follow `recipes/create-new-worker.md` |
| "build a Go daemon for Linux" | Use `templates/linux-agent/`; add systemd via `recipes/add-docker-systemd.md` |
| "build a scheduler/cron in Go" | Use `templates/scheduler/` |
| "build a webhook receiver" | Use `templates/webhook-service/`; follow `recipes/create-new-webhook-service.md` |
| "build a gRPC service" | Use `templates/grpc-service/` |
| "build an MCP backend in Go" | Use `templates/rest-api/` + observability via `recipes/add-observability.md` |
| "build an agent runtime in Go" | Use `templates/linux-agent/` + graceful shutdown + observability |
| "improve my Go repo" | Run `scripts/audit_go_service.sh`; follow `recipes/improve-existing-repo.md` |
| "make my Go service production-ready" | Follow `recipes/prepare-for-production.md` |
| "add tests to my Go project" | Follow `recipes/add-tests.md` |
| "review my Go code" | Use `review/review-prompt.md` with `go-service-scorecard.md` |
| "walk through building X" | Use `examples/` walkthroughs (e.g., `examples/01-build-rest-api.md`) |

**Deactivation:** Do not activate if the user explicitly says "don't use this skill," "it's just a quick hack," or "learning Go." If unsure, activate — the skill cost is low.

---

## 3. Agent Operating Rules

Every agent working on a Go project must follow these rules without exception.

### 3.1 Inspect Before Changing

Before writing any code in an existing repository:

```bash
# 1. Check git state
git status --short
git branch --show-current

# 2. Find the module path
head -3 go.mod  # Module name is in line 2

# 3. Check Go version
go version

# 4. Check project structure
find . -maxdepth 3 -type f -name "*.go" | head -20
find . -maxdepth 2 -type d | sort

# 5. Check for existing tests
find . -name "*_test.go" | wc -l

# 6. Check Docker/Compose files
ls -la Dockerfile* docker-compose* 2>/dev/null || true
```

### 3.2 Detect Module Path

Always use the **correct module path** from `go.mod`. Never guess.

```
# From go.mod line 2:
module github.com/myorg/myproject

# Correct import:
import "github.com/myorg/myproject/internal/service"
```

### 3.3 Detect Go Version

Check `go.mod` for the minimum Go version:

```
go 1.22
```

Use only features available in that version. Default to Go 1.22+ features (slog, slices.Concat, maps etc.).

### 3.4 Project Structure Rules

**AVOID** single-file production services. A production service requires:

```
myproject/
├── cmd/           # ONE SUBDIR PER BINARY (never main.go in root)
│   └── myservice/
│       └── main.go
├── internal/      # Private application code (not importable)
│   ├── config/
│   ├── domain/
│   ├── service/
│   ├── repository/
│   ├── handler/
│   └── infrastructure/
├── pkg/           # Only for truly public libraries
├── api/           # OpenAPI specs, proto files
├── migrations/    # SQL migrations (goose)
├── configs/      # Config templates (.tmpl)
├── scripts/       # Build/deployment scripts
├── Dockerfile
├── docker-compose.yml
├── Makefile
└── go.mod
```

### 3.5 Context.Context Rules

Every function doing I/O, waiting, or anything that can timeout MUST accept `context.Context`:

```go
// CORRECT
func (s *UserService) GetUser(ctx context.Context, id int64) (*User, error)

// WRONG - missing context
func (s *UserService) GetUser(id int64) (*User, error)

// NEVER store context in struct fields
type BadService struct {
    ctx context.Context  // WRONG
    db  *sql.DB
}
```

Context rules:
- `context.Context` is always the **first parameter**
- Never pass `context.Background()` to deep call chains — let the caller provide it
- Always wrap with timeout for external calls: `context.WithTimeout(ctx, 5*time.Second)`
- Check `ctx.Done()` in loops

### 3.6 Graceful Shutdown Rules

Every long-running service MUST handle SIGTERM and SIGINT:

```go
func main() {
    srv := &http.Server{Addr: ":8080", Handler: mux}

    go func() {
        if err := srv.ListenAndServe(); err != nil {
            slog.Error("server error", "error", err)
        }
    }()

    // Wait for signal
    sig := make(chan os.Signal, 1)
    signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)
    <-sig

    // Graceful shutdown with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    srv.Shutdown(ctx)
    slog.Info("server stopped cleanly")
}
```

Shutdown requirements:
- HTTP servers: drain in-flight requests (use `srv.Shutdown(ctx)`)
- Workers: finish current job or cancel
- Queue consumers: finish current message before stopping
- Database connections: close properly

### 3.7 Health and Readiness Endpoints

Every HTTP service requires:

```go
// Liveness - is the process alive?
func healthHandler(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(map[string]string{"status": "ok"})
}

// Readiness - can the service serve traffic?
func readyHandler(w http.ResponseWriter, r *http.Request) {
    if err := db.PingContext(r.Context()); err != nil {
        http.Error(w, "db unhealthy", http.StatusServiceUnavailable)
        return
    }
    w.WriteHeader(http.StatusOK)
}

// Routes
mux.HandleFunc("GET /health", healthHandler)
mux.HandleFunc("GET /ready", readyHandler)
```

### 3.8 Structured Logging Rules

**NEVER** use `fmt.Printf`, `log.Print`, or `log.Println` in production code.

```go
import "log/slog"

// CORRECT - structured with fields
slog.Info("user created",
    "user_id", user.ID,
    "email", user.Email,
    "role", user.Role)

// CORRECT - JSON in production
handler := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.LevelInfo,
})

// WRONG - unstructured
slog.Info("user created")  // No context!

// WRONG - fmt
fmt.Printf("user %d created", user.ID)
```

Log levels:
- `DEBUG`: Detailed diagnostic info (dev only)
- `INFO`: Normal operational events
- `WARN`: Degraded performance, approaching limits
- `ERROR`: Failures needing attention

**NEVER log secrets**: tokens, passwords, API keys, bearer tokens.

### 3.9 Avoid Global Mutable State

```go
// WRONG - global mutable state
var db *sql.DB
var cache map[string]string

// CORRECT - pass dependencies
func main() {
    db, _ := database.Open(cfg)
    handler := NewUserHandler(db)  // Dependency injection
}
```

If you need shared state, use:
- `sync.Mutex` for simple shared values
- `sync.RWMutex` for read-heavy workloads
- `channel` for communication between goroutines
- **Never** use package-level vars for connections or caches

### 3.10 Avoid Panic in Request/Worker Paths

```go
// WRONG - panic on normal error
func (h *Handler) GetUser(w http.ResponseWriter, r *http.Request) {
    user, err := h.svc.GetUser(r.Context(), id)
    if err != nil {
        panic(err)  // CRASHES THE SERVER
    }
}

// CORRECT - return error
func (h *Handler) GetUser(w http.ResponseWriter, r *http.Request) {
    user, err := h.svc.GetUser(r.Context(), id)
    if err != nil {
        handleError(w, err)  // Graceful error handling
        return
    }
}
```

**Allowed uses of panic:**
- Unrecoverable startup errors (DB connection impossible, required env missing)
- Programming errors that indicate bugs (switch exhaustive check default)

### 3.11 Avoid Goroutine Leaks

Every goroutine must have a clear termination path:

```go
// WRONG - goroutine with no exit
go func() {
    for {
        msg := queue.Receive()
        process(msg)
    }
}()

// CORRECT - goroutine with context and exit
func worker(ctx context.Context, queue Queue) {
    for {
        select {
        case <-ctx.Done():
            return  // Clean exit
        case msg := <-queue:
            process(ctx, msg)
        }
    }
}

// CORRECT - WaitGroup for tracked workers
var wg sync.WaitGroup
for i := 0; i < 3; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        worker(ctx)
    }()
}
wg.Wait()  // Block until all done
```

### 3.12 Avoid Swallowing Errors

```go
// WRONG - silent ignore
_, err := db.ExecContext(ctx, query)
_ = err  // SILENTLY IGNORED

// WRONG - losing error chain
user, _ := s.repo.GetByID(ctx, id)  // If err, user is nil, no error reported

// CORRECT - handle or return
if err := db.ExecContext(ctx, query); err != nil {
    return fmt.Errorf(" ExecContext: %w", err)
}

// CORRECT - check return
user, err := s.repo.GetByID(ctx, id)
if err != nil {
    return fmt.Errorf("GetUser: %w", err)
}
```

Error wrapping format: `fmt.Errorf("operation: %w", err)`

### 3.13 Write Tests for Critical Logic

At minimum:
- Service layer: >70% coverage
- Handler layer: >60% coverage
- Domain logic: >80% coverage

```bash
# Run tests with race detector
go test -race -coverprofile=coverage.out ./...
```

### 3.14 Document Commands and Environment Variables

Every service needs:

1. **README.md** with:
   - Quick start commands
   - Environment variables table
   - Architecture description

2. **.env.example** (no secrets):
   ```
   DATABASE_URL=postgresql://user:password@localhost:5432/mydb
   REDIS_ADDR=localhost:6379
   LOG_LEVEL=info
   SERVER_PORT=8080
   ```

3. **Makefile** with:
   ```
   build, test, lint, clean, run, docker-build, docker-up
   ```

---

## 4. Project Layout Rules

### cmd/
```
cmd/
├── myservice/     # One subdir per binary
│   └── main.go    # Entry point. Imports internal packages ONLY.
└── cli/
    └── main.go
```

Rules:
- **Never** put `main.go` in the root or any directory outside `cmd/`
- `main.go` should be thin: parse config, setup logger, start services, block on shutdown
- All business logic lives in `internal/`

### internal/
```
internal/
├── config/        # Configuration loading from env
├── domain/        # Entities, interfaces, domain errors (zero external deps)
├── service/       # Business logic (depends on domain, not infrastructure)
├── repository/    # Data access interfaces (defined here, implemented in infrastructure)
├── handler/       # HTTP/gRPC handlers (decode request, call service, encode response)
├── middleware/    # HTTP middleware (logging, recovery, CORS, rate limiting)
└── infrastructure/ # External adapters (DB, cache, queue, external APIs)
```

**Critical**: `internal/` packages cannot be imported by external modules. This is enforced by Go tooling.

### pkg/
Use `pkg/` only for packages that are:
- Truly public and reusable by other projects
- Well-documented APIs

Do NOT use `pkg/` as a "shared utils" dumping ground. If only your project uses it, it belongs in `internal/`.

### api/
```
api/
├── openapi.yaml        # OpenAPI 3.0 spec
└── v1/
    └── service.proto   # gRPC proto files
```

### migrations/
```
migrations/
├── 001_create_users.sql
├── 002_add_sessions.sql
└── 003_alter_users_roles.sql
```

Use [goose](https://github.com/pressly/goose) for migrations. Prefix files with sequence number.

### configs/
Configuration file templates (not to be confused with `internal/config/`):

```
configs/
├── service.yaml.tmpl    # Template with {{.Port}} placeholders
└── nginx.conf.tmpl
```

Render these with `text/template` or include in deployments.

### deployments/
Container and orchestration manifests:

```
deployments/
├── docker/
│   └── Dockerfile
├── kubernetes/
│   ├── deployment.yaml
│   └── service.yaml
├── systemd/
│   └── myservice.service
└── docker-compose.yml
```

### scripts/
Build, deployment, and utility scripts:

```
scripts/
├── build.sh
├── migrate.sh
└── seed.sh
```

### docs/
Architecture decision records and API documentation:

```
docs/
├── architecture/
│   └── adrs/
│       ├── 001ADR-logging.md
│       └── 002ADR-database.md
└── api/
    └── reference.md
```

### test/
Shared test utilities and fixtures:

```
test/
├── fixtures/
├── testdb/
└── asserts.go
```

---

## 5. Framework Decision Rules

### HTTP Routing

| Framework | When to Use |
|-----------|------------|
| **stdlib `net/http` ServeMux** | Simple APIs, low complexity |
| **`chi`** | Need path parameters, middleware, better routing |
| **`gorilla/mux`** | Need advanced routing, websockets |
| **`gin`** | HIGH TRAFFIC APIs needing maximum performance |
| **`fiber`** | Extreme performance needs (check maintenance status) |

**Default choice**: stdlib `ServeMux` for <10 endpoints, `chi` for everything else.

**Avoid**: `gin` and `fiber` unless you have specific performance requirements — they add complexity and maintenance burden.

### gRPC

Use [google.golang.org/grpc](https://pkg.go.dev/google.golang.org/grpc) for:
- Internal service-to-service communication
- High-performance APIs
- Bidirectional streaming

Generate code with `protoc` + `protoc-gen-go` + `protoc-gen-go-grpc`.

### CLI

| Library | When to Use |
|---------|------------|
| **stdlib `flag`** | Simple CLIs with few flags |
| **`cobra`** | Complex CLIs with subcommands |
| **`urfave/cli`** | Simpler alternative to cobra |

**Default choice**: `cobra` for anything beyond basic flag parsing.

### Database

| Tool | Purpose |
|------|---------|
| **`jackc/pgx`** | PostgreSQL driver with connection pooling |
| **`sqlc`** | Generate type-safe Go from SQL (PREFERRED) |
| **`goose`** | Database migrations |
| **`sqlx`** | Only if sqlc is insufficient |

**Never use**: GORM (hidden SQL, performance issues, magic behavior).

### Logging

| Library | When to Use |
|---------|------------|
| **`log/slog`** (stdlib, Go 1.21+) | **DEFAULT** — structured logging in stdlib |
| **`uber-go/zap`** | Ultra-low-latency services needing zero-allocation logging |

**Default**: `slog` with JSON handler for production.

### Configuration

| Library | When to Use |
|---------|------------|
| **`os.Getenv` + manual parsing** | Simple config, few vars |
| **`envconfig`** | Structured config from env vars |
| **`viper`** | Need YAML/JSON/TOML config + env override |

**Default**: Direct `os.Getenv` calls with a typed config struct.

### Metrics

| Library | When to Use |
|---------|------------|
| **`prometheus/client_golang`** | Prometheus metrics (DEFAULT) |

### Tracing

| Library | When to Use |
|---------|------------|
| **`go.opentelemetry.io/otel`** | Distributed tracing (DEFAULT for microservices) |

### Framework Summary Table

| Concern | Recommended | Alternative |
|---------|-------------|-------------|
| HTTP routing | stdlib / chi | gorilla/mux, gin |
| gRPC | google.golang.org/grpc | — |
| CLI | cobra | flag, urfave/cli |
| Database driver | jackc/pgx | — |
| Type-safe queries | sqlc | — |
| Migrations | goose | — |
| Logging | slog | zap |
| Config | os.Getenv + struct | envconfig, viper |
| Metrics | prometheus/client_golang | — |
| Tracing | opentelemetry | — |
| Validation | go-playground/validator | — |
| JWT | golang-jwt/jwt | — |
| Redis | redis/go-redis | — |

---

## 6. Generation Workflow

When building a new Go project, follow this sequence:

### Step 1: Understand Requirements
- What type of service? (API, CLI, worker, scheduler, agent)
- What are the external dependencies? (DB, queue, cache)
- What is the deployment target? (Docker, systemd, K8s)
- What are the non-functional requirements? (latency, throughput, availability)

### Step 2: Choose Template
Use `scripts/scaffold_go_service.sh` or manually select from `templates/`:

```bash
# Scaffold a REST API
./scripts/scaffold_go_service.sh rest-api myapi --output-dir ./services

# Scaffold a worker daemon
./scripts/scaffold_go_service.sh worker-daemon myworker --output-dir ./services
```

Available templates:
- `cli` — CLI tool with subcommands
- `rest-api` — HTTP REST API with handlers and middleware
- `worker-daemon` — Long-running worker with queue consumer
- `grpc-service` — gRPC service with protobuf
- `webhook-service` — Webhook receiver with signature verification
- `scheduler` — Cron-style scheduled jobs
- `linux-agent` — Linux daemon with systemd unit

### Step 3: Scaffold Project
```bash
mkdir -p myproject
cd myproject
go mod init github.com/myorg/myproject
```

Create directory structure:
```bash
mkdir -p cmd/myservice internal/{config,domain,service,repository,handler,middleware,infrastructure}
mkdir -p migrations configs deployments/scripts
touch go.mod go.mod
```

### Step 4: Wire Configuration

`internal/config/config.go`:
```go
type Config struct {
    Server   ServerConfig
    Database DatabaseConfig
    Logging  LoggingConfig
}

type ServerConfig struct {
    Port string
}

type DatabaseConfig struct {
    URL string
}

func Load() *Config {
    return &Config{
        Server: ServerConfig{
            Port: getEnv("SERVER_PORT", "8080"),
        },
        Database: DatabaseConfig{
            URL: os.Getenv("DATABASE_URL"),
        },
        Logging: LoggingConfig{
            Level:  getEnv("LOG_LEVEL", "info"),
            Format: getEnv("LOG_FORMAT", "json"),
        },
    }
}
```

### Step 5: Wire Logger

```go
func setupLogger(cfg config.LoggingConfig) *slog.Logger {
    opts := &slog.HandlerOptions{Level: parseLevel(cfg.Level)}
    var handler slog.Handler
    if cfg.Format == "json" {
        handler = slog.NewJSONHandler(os.Stdout, opts)
    } else {
        handler = slog.NewTextHandler(os.Stdout, opts)
    }
    return slog.New(handler)
}
```

### Step 6: Wire Context and Shutdown

```go
func main() {
    cfg := config.Load()
    logger := setupLogger(cfg.Logging)
    slog.SetDefault(logger)

    // Context for graceful shutdown
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    // Setup server with shutdown
    srv := &http.Server{Addr: ":" + cfg.Server.Port, Handler: mux}

    go func() {
        if err := srv.ListenAndServe(); err != nil {
            slog.Error("server error", "error", err)
        }
    }()

    // Wait for signal
    sig := make(chan os.Signal, 1)
    signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)
    <-sig

    slog.Info("shutting down")

    // Graceful shutdown with timeout
    shutdownCtx, shutdownCancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer shutdownCancel()

    srv.Shutdown(shutdownCtx)
}
```

### Step 7: Implement Domain Layer

`internal/domain/user.go`:
```go
type User struct {
    ID        int64
    Email     string
    Name      string
    Role      Role
    CreatedAt time.Time
}

type Role string

const (
    RoleAdmin  Role = "admin"
    RoleUser   Role = "user"
)
```

`internal/domain/errors.go`:
```go
var (
    ErrNotFound      = errors.New("entity not found")
    ErrAlreadyExists = errors.New("entity already exists")
    ErrInvalidInput  = errors.New("invalid input")
)
```

### Step 8: Implement Repository Interface (in domain)

`internal/domain/user_repo.go`:
```go
type UserRepository interface {
    Create(ctx context.Context, user *User) error
    GetByID(ctx context.Context, id int64) (*User, error)
    List(ctx context.Context, limit, offset int) ([]*User, error)
}
```

### Step 9: Implement Service Layer

`internal/service/user_service.go`:
```go
type UserService struct {
    repo domain.UserRepository
}

func NewUserService(repo domain.UserRepository) *UserService {
    return &UserService{repo: repo}
}

func (s *UserService) GetUser(ctx context.Context, id int64) (*domain.User, error) {
    user, err := s.repo.GetByID(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("GetUser: %w", err)
    }
    return user, nil
}
```

### Step 10: Implement Repository Implementation (in infrastructure)

`internal/infrastructure/database/user_repo.go`:
```go
type UserRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) *UserRepository {
    return &UserRepository{db: db}
}

func (r *UserRepository) GetByID(ctx context.Context, id int64) (*domain.User, error) {
    query := `SELECT id, email, name, role, created_at FROM users WHERE id = $1`
    var user domain.User
    err := r.db.QueryRowContext(ctx, query, id).Scan(&user.ID, &user.Email, &user.Name, &user.Role, &user.CreatedAt)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, domain.ErrNotFound
        }
        return nil, fmt.Errorf("QueryRowContext: %w", err)
    }
    return &user, nil
}
```

### Step 11: Implement Handlers

`internal/handler/user_handler.go`:
```go
type UserHandler struct {
    svc *service.UserService
}

func NewUserHandler(svc *service.UserService) *UserHandler {
    return &UserHandler{svc: svc}
}

func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {
    id, err := strconv.ParseInt(r.PathValue("id"), 10, 64)
    if err != nil {
        http.Error(w, "invalid id", http.StatusBadRequest)
        return
    }

    user, err := h.svc.GetUser(r.Context(), id)
    if err != nil {
        handleError(w, err)
        return
    }

    json.NewEncoder(w).Encode(toUserResponse(user))
}
```

### Step 12: Add Tests

`internal/service/user_service_test.go`:
```go
type mockUserRepository struct {
    users map[int64]*domain.User
}

func (m *mockUserRepository) GetByID(ctx context.Context, id int64) (*domain.User, error) {
    user, ok := m.users[id]
    if !ok {
        return nil, domain.ErrNotFound
    }
    return user, nil
}

func TestUserService_GetUser(t *testing.T) {
    mock := &mockUserRepository{users: map[int64]*domain.User{
        1: {ID: 1, Email: "test@example.com"},
    }}
    svc := NewUserService(mock)

    user, err := svc.GetUser(context.Background(), 1)
    if err != nil {
        t.Fatalf("GetUser() error = %v", err)
    }
    if user.Email != "test@example.com" {
        t.Errorf("email = %s, want test@example.com", user.Email)
    }
}
```

### Step 13: Add Dockerfile

```dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -ldflags="-w -s" -o service ./cmd/myservice

FROM alpine:3.19
RUN apk add --no-cache ca-certificates && adduser -D -u 1000 appuser
COPY --from=builder /app/service /app/service
USER 1000
ENTRYPOINT ["/app/service"]
```

### Step 14: Add systemd Unit (for daemons)

`deployments/systemd/myservice.service`:
```ini
[Unit]
Description=My Service
After=network.target

[Service]
Type=simple
User=myservice
WorkingDirectory=/opt/myservice
ExecStart=/opt/myservice/myservice serve
TimeoutStopSec=30
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### Step 15: Add README

```markdown
# myservice

REST API for [purpose].

## Quick Start

```bash
go build -o myservice ./cmd/myservice
DATABASE_URL=postgresql://... ./myservice serve
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| DATABASE_URL | PostgreSQL connection string | (required) |
| SERVER_PORT | HTTP port | 8080 |
| LOG_LEVEL | debug, info, warn, error | info |

## Endpoints

- `GET /health` — Liveness check
- `GET /ready` — Readiness check
- `GET /api/v1/users/:id` — Get user
```

### Step 16: Run Validation

```bash
# Format and vet
go fmt ./...
go vet ./...

# Test with race detector
go test -race ./...

# Build for Linux
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o myservice ./cmd/myservice

# Validate structure
../goops-forge-skill/scripts/validate_go_project.sh .
```

---

## 7. Existing Repo Improvement Workflow

When improving an existing repository:

### Step 1: Map Architecture

Before changing anything, understand the current state:

```bash
# List all Go files by directory
find . -name "*.go" -not -path "./.git/*" | \
  sed 's|/[^/]*$||' | sort -u | \
  awk -F'/' '{print $2 "/" $3}' | sort -u | grep -v "^$"

# Find main entry points
find . -name "main.go" -not -path "./.git/*"

# Check current module path
head -2 go.mod

# Find all external dependencies
grep -h "import" --include="*.go" . | sort -u
```

### Step 2: Identify Risks

Ask:
- Does it compile with `go vet`?
- Are there tests? Do they pass with `-race`?
- Is there graceful shutdown?
- Is there structured logging?
- Is config from env?
- Are there obvious security issues?

Run: `../goops-forge-skill/scripts/audit_go_service.sh .`

### Step 3: Produce Changes in Small Commits

**Never rewrite everything.** Break improvements into logical commits:

```
commit 1: add config loading from env
commit 2: replace log.Print with slog
commit 3: add context to service methods
commit 4: add graceful shutdown
commit 5: add health endpoints
commit 6: add tests for service layer
```

### Step 4: Avoid Rewriting Everything

Fix what's broken. Don't "improve" working code without cause.

- If it works and has tests, don't touch it
- If it works but has no tests, add tests (don't rewrite)
- If it doesn't work, fix only the broken parts

### Step 5: Preserve Public API Unless Explicitly Requested

If the service has existing API consumers:
- Maintain backward compatibility
- Add deprecation warnings
- Version the API if breaking changes are needed

### Step 6: Add Migration Notes

If changing configs, database schema, or environment variables:
- Document in README
- Add to CHANGELOG
- Provide migration steps

---

## 8. Quality Gates

Every Go project must pass these gates before production deployment:

### Gate 1: Format
```bash
go fmt ./...
```
Must pass with no changes.

### Gate 2: Vet
```bash
go vet ./...
```
Must pass with no warnings.

### Gate 3: Static Analysis (if available)
```bash
go install honnef.co/go/tools/cmd/staticcheck@latest
staticcheck ./...
```
Fix all reported issues or document why an issue is a false positive.

### Gate 4: Tests
```bash
go test -v -race ./...
```
All tests must pass. The `-race` flag is mandatory to catch race conditions.

### Gate 5: Test Coverage
```bash
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out | grep total
```
Service and domain layers: >70% coverage. Handlers: >60%.

### Gate 6: Build
```bash
# Native build
go build -o /dev/null ./...

# Cross-compile for Linux
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o myservice ./cmd/myservice
```
Must compile without errors.

### Gate 7: Docker Build (if applicable)
```bash
docker build -t myservice:test .
```
Must produce a working image.

### Gate 8: Health Endpoint
```bash
curl http://localhost:8080/health
# Expected: {"status":"ok"}
```

### Gate 9: Readiness Endpoint
```bash
curl http://localhost:8080/ready
# Expected: {"status":"ready"} or 503 if not ready
```

### Gate 10: Config Validation
Start the service without required env vars — it must fail with a clear error message, not hang or crash obscurely.

### Gate 11: Graceful Shutdown Test
Send SIGTERM to the running process — it must:
1. Stop accepting new connections
2. Complete in-flight requests (up to timeout)
3. Exit cleanly

### Gate 12: Validation Script
```bash
./goops-forge-skill/scripts/validate_go_project.sh .
```

---

## 9. Anti-Patterns

### Anti-Pattern: main.go Does Everything

```go
// WRONG - main.go contains config, DB, handlers, everything
func main() {
    db, _ := sql.Open(...)
    http.HandleFunc("/", func(w, r) {
        // DB queries, business logic, everything inline
    })
    http.ListenAndServe(":8080", nil)
}

// CORRECT - main.go is thin, delegates to packages
func main() {
    cfg := config.Load()
    db, _ := database.Open(cfg.Database)
    svc := service.NewUserService(db)
    handler := handler.NewUserHandler(svc)
    // ... setup routes, start server
}
```

### Anti-Pattern: context.Background() Everywhere

```go
// WRONG - context.Background() in deep call chain loses cancellation
go processAll(context.Background(), items)

// CORRECT - propagate context from caller
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    go processAll(ctx, items)
}
```

### Anti-Pattern: log.Println Scattered Everywhere

```go
// WRONG - unstructured, no fields
log.Println("user created")
fmt.Printf("processing item %d\n", item.ID)

// CORRECT - structured with fields
slog.Info("user created", "user_id", user.ID, "email", user.Email)
```

### Anti-Pattern: Panic for Normal Errors

```go
// WRONG - panic crashes the server
if user == nil {
    panic("user not found")
}

// CORRECT - return error
if user == nil {
    return nil, ErrNotFound
}
```

### Anti-Pattern: Unbounded Goroutines

```go
// WRONG - goroutine leak on shutdown
for _, item := range items {
    go process(item)  // No way to wait for these
}

// CORRECT - bounded worker pool
pool := worker.NewPool(workers, queue, processor)
pool.Start(ctx)
pool.Stop(timeout)
```

### Anti-Pattern: Hardcoded Env/Secrets

```go
// WRONG - hardcoded
db, _ := sql.Open("postgres", "user=admin password=secret")

// CORRECT - from environment
db, _ := sql.Open("postgres", os.Getenv("DATABASE_URL"))
```

### Anti-Pattern: No Timeout on HTTP Clients

```go
// WRONG - infinite timeout
client := &http.Client{}

// CORRECT - timeout on requests
client := &http.Client{
    Timeout: 30 * time.Second,
}
```

### Anti-Pattern: No Shutdown Path

```go
// WRONG - no graceful shutdown, hard crash on SIGTERM
func main() {
    srv := &http.Server{Addr: ":8080"}
    srv.ListenAndServe()  // SIGTERM just kills it
}

// CORRECT - graceful shutdown with timeout
func main() {
    srv.ListenAndServe()
    // In goroutine, then:
    srv.Shutdown(30 * time.Second)
}
```

### Anti-Pattern: No Migration System

```go
// WRONG - manual schema management
// "just run this SQL manually"

// CORRECT - use goose migrations
goose postgres $DATABASE_URL up
```

### Anti-Pattern: No Tests

Production code without tests is a liability, not an asset.

```bash
# WRONG - "it compiles, ship it"
# CORRECT:
go test -race ./...
```

### Anti-Pattern: Unclear Package Boundaries

```go
// WRONG - everything in one package
package main  // 2000 lines, everything mixed

// CORRECT - clear separation
package domain   // entities, interfaces
package service  // business logic
package handler  // HTTP layer
```

---

## 10. Output Contract

When an agent completes a Go project using this skill, the output must include:

### Required Outputs

| Output | Description | Location |
|--------|-------------|----------|
| **Working code** | Compiles, starts, serves requests | `cmd/`, `internal/` |
| **README.md** | Setup instructions, env vars, architecture | `./README.md` |
| **.env.example** | All environment variables documented | `./.env.example` |
| **Makefile** | Standard targets: build, test, lint, clean, run | `./Makefile` |
| **Dockerfile** | Multi-stage build, non-root user, minimal image | `./Dockerfile` |
| **Tests** | Unit tests for service/domain layers | `*_test.go` files |
| **Graceful shutdown** | SIGTERM/SIGINT handling | `cmd/*/main.go` |
| **Health endpoints** | `/health` and `/ready` | handler |
| **Structured logging** | slog with JSON format | `main.go` |

### Optional Outputs (when applicable)

| Output | When Required |
|--------|--------------|
| **docker-compose.yml** | Multiple services or dependencies |
| **systemd unit file** | Linux daemon deployment |
| **Prometheus metrics** | Production services needing observability |
| **OpenTelemetry tracing** | Distributed/microservice architecture |
| **API documentation** | Public REST APIs |

### Validation Report

After completing a project, produce a validation report:

```markdown
## Validation Report

| Gate | Command | Result |
|------|---------|--------|
| go fmt | `go fmt ./...` | PASS |
| go vet | `go vet ./...` | PASS |
| Tests | `go test -race ./...` | PASS (47/47) |
| Build | `GOOS=linux go build` | PASS |
| Docker | `docker build` | PASS |
| Health | `curl /health` | PASS |
| Config | Missing DATABASE_URL | Fails with clear error |

### Coverage

| Layer | Coverage |
|-------|----------|
| Domain | 85% |
| Service | 78% |
| Handler | 65% |
| Infrastructure | 52% |

### Known Limitations

- Integration tests require external DB (not run in CI)
- Load testing not performed

### Verdict: PRODUCTION_READY
```

---

## File Index

```
goops-forge-skill/
├── SKILL.md                      ← THIS FILE — skill definition and routing
├── README.md                      ← Package overview and installation
├── guides/
│   ├── 01-go-project-layout.md      ← Layout rules
│   ├── 02-service-architecture.md     ← Layered architecture
│   ├── 03-context-cancellation.md    ← Context patterns
│   ├── 04-graceful-shutdown.md      ← Shutdown requirements
│   ├── 05-config-env-management.md   ← Configuration from env
│   ├── 06-structured-logging.md      ← Logging standards (slog)
│   ├── 07-http-api-patterns.md       ← HTTP handler patterns
│   ├── 08-worker-queue-patterns.md   ← Worker and queue patterns
│   ├── 09-database-sqlc-goose.md    ← Database workflow
│   ├── 10-testing-quality.md         ← Testing standards
│   ├── 11-observability.md           ← Metrics and tracing
│   ├── 12-docker-linux-systemd.md    ← Container and systemd
│   ├── 13-security-hardening.md     ← Security practices
│   └── 14-agent-review-rules.md     ← Code review for agents
├── templates/
│   ├── cli/                  ← CLI tool template
│   ├── rest-api/             ← REST API template
│   ├── worker-daemon/         ← Worker daemon template
│   ├── grpc-service/         ← gRPC service template
│   ├── webhook-service/      ← Webhook receiver template
│   ├── scheduler/            ← Scheduled job template
│   └── linux-agent/          ← Linux agent template
├── scripts/
│   ├── scaffold_go_service.sh    ← Project scaffolder
│   ├── validate_go_project.sh    ← Validation script
│   └── audit_go_service.sh       ← Security audit script
├── checklists/
│   ├── production-readiness.md    ← Pre-deployment checklist
│   ├── security-checklist.md      ← Security hardening
│   ├── observability-checklist.md ← Metrics and logging
│   ├── testing-checklist.md       ← Test coverage requirements
│   └── linux-deploy-checklist.md  ← Linux deployment steps
├── decisions/
│   ├── 001-layout-rules.md                ← ADR: project layout
│   ├── 002-framework-selection.md        ← ADR: library choices
│   ├── 003-logging-config-db-rules.md     ← ADR: logging/config/DB
│   └── 004-agent-safe-generation-rules.md ← ADR: agent generation rules
├── examples/                ← Practical walkthroughs (for agents and humans)
│   ├── 01-build-rest-api.md
│   ├── 02-build-worker-daemon.md
│   ├── 03-build-linux-agent.md
│   ├── 04-review-existing-go-repo.md
│   ├── 05-harden-basic-go-server.md
│   ├── 06-add-observability.md
│   ├── 07-add-systemd-deployment.md
│   └── 08-add-sqlc-goose-database.md
├── recipes/                 ← Focused task workflows
│   ├── README.md
│   ├── create-new-cli.md
│   ├── create-new-api-service.md
│   ├── create-new-worker.md
│   ├── create-new-webhook-service.md
│   ├── improve-existing-repo.md
│   ├── add-docker-systemd.md
│   ├── add-tests.md
│   ├── add-observability.md
│   └── prepare-for-production.md
└── tests/
    └── README.md
```

---

## Critical Rules Summary

1. **No goroutine leaks** — every goroutine has a termination path
2. **Context is mandatory** — first param for all cancellable operations
3. **Structured logging** — slog with fields, no fmt.Printf
4. **Config from env** — required fields validated at startup
5. **Graceful shutdown** — SIGTERM/SIGINT with timeout
6. **No secrets in logs** — scrub tokens, passwords, keys
7. **No panic in request/worker paths** — return errors
8. **Error wrapping** — `fmt.Errorf("operation: %w", err)`
9. **No silent error ignores** — `_ = err` is forbidden
10. **Test critical logic** — service/domain layers need >70% coverage

---
> Source: [Arvuno/GO-ops-Forge-SKill](https://github.com/Arvuno/GO-ops-Forge-SKill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
