## flowra

> `AGENTS.md` is the single source of contributor guidance for this repository.

# Repository Guidelines

## Context Requirements
`AGENTS.md` is the single source of contributor guidance for this repository.

## Project Overview

Flowra is a chat-centric collaboration system with integrated task management and help desk capabilities.

Core stack:
- Backend: Go 1.26+ with Echo v4.
- Data: MongoDB 6+ (Go Driver v2) + Redis.
- Frontend: HTMX 2+, Pico CSS v2, vanilla JS modules.
- Auth: Keycloak SSO.
- Runtime: Docker Compose for local/dev and self-hosted helpers.

## Project Structure & Module Organization
`flowra` is a Go monorepo using clean architecture with event sourcing:

- **`cmd/`**: Application entry points (`api`, `worker`, `tools`)
- **`internal/`**: Private application code organized by layer:
  - `domain/`: Aggregates, entities, events, domain logic (6 aggregates, 30+ events)
  - `application/`: Use cases and business workflows (40+ use cases)
  - `infrastructure/`: External dependencies (MongoDB, Redis, Keycloak, EventStore)
  - `handler/http/`: HTTP request handlers (REST + HTMX endpoints)
  - `handler/websocket/`: WebSocket handlers for real-time updates
  - `middleware/`: HTTP middleware (auth, CORS, logging, rate limiting)
  - `service/`: Business services (workspace access, chat, member, auth)
  - `worker/`: Background workers (user sync)
- **`web/`**: Frontend (HTMX + Pico CSS templates, static assets)
- **`tests/`**: Test suites organized by scope:
  - `integration/`: Integration tests with real infrastructure (testcontainers)
  - `e2e/`: End-to-end tests (API and frontend browser tests)
  - `e2e/frontend/`: Playwright-based browser E2E tests
  - `load/`: Manual load tests (k6 scripts)
  - `mocks/`: Shared mock implementations
  - `testutil/`: Test utilities and helpers
- **`configs/`**: Configuration files
- **`docs/`**: Documentation (architecture, API specs, guides)

## Build, Test, and Development Commands

### Development
```bash
# Start full development environment (recommended)
make dev                    # Docker infra + worker + API

# Start API only (no worker, limited features)
make dev-lite              # FLOWRA_DEV_MODE=lite go run ./cmd/api

# Build binaries
make build                 # Creates bin/api and bin/worker

# Manage infrastructure
make docker-up            # Start MongoDB, Redis, Keycloak
make docker-down          # Stop all services
make reset-data           # Reset Chat=SoT data (when switching branches)
```

### Testing
```bash
# Run all tests
make test                             # Full suite with race detector

# Run specific test types
make test-unit                        # Unit tests: go test ./internal/...
make test-integration                 # Integration: -tags=integration
make test-e2e                         # E2E API: -tags=e2e
make test-e2e-frontend                # Browser E2E: -tags=e2e ./tests/e2e/frontend/...
make test-e2e-frontend-smoke          # Quick smoke test for board/sidebar

# Run a single test
go test -v ./internal/domain/chat -run TestChat_NewChat
go test -tags=integration -v ./tests/integration -run TestChatSoT

# Generate coverage
make test-coverage                    # Generates coverage.html

# Load testing (manual, requires k6 and AUTH_TOKEN)
make test-load-tags
```

### Code Quality
```bash
# Format and lint (ALWAYS run before committing)
make lint                  # go fmt + golangci-lint --fix

# Dependencies
make deps                  # go mod download && go mod tidy

# Playwright setup (for frontend E2E)
make playwright-install    # Install Chromium browser

# Self-hosted/production helpers
make docker-build          # Build production image
make docker-prod-up        # Start self-hosted production stack
make docker-prod-logs      # Tail production stack logs
make docker-prod-down      # Stop production stack
```

## Runtime & Access

- Main local app URL: `http://localhost:8080`
- API docs: `http://localhost:8080/docs`
- Health: `http://localhost:8080/health`
- Keycloak admin: `http://localhost:8090` (`admin/admin123`)
- Test user login: `testuser` / `test123`

Runtime toggles for API binary:
- `FLOWRA_WORKER=true` enables unified API + worker loops.
- `FLOWRA_WORKER=false` runs API-only mode.
- `--with-worker` / `--with-worker=false` overrides env value.

When switching branches around Chat=SoT changes, run `make reset-data` after bringing infra up.

## Configuration & Indexes

- Main configs: `configs/config.yaml`, `configs/config.dev.yaml`, `configs/config.prod.yaml`
- Local runtime stack: `docker-compose.yml`
- Self-hosted stack: `docker-compose.prod.yml`
- MongoDB indexes are managed in code at `internal/infrastructure/mongodb/indexes.go`
- Indexes are created on API startup and in integration test setup (no standalone migration runner required)

## Chat=SoT Guardrails

For typed entities (`task`, `bug`, `epic`) follow ADR-007 (`docs/architecture/adr-007-chat-sot.md`):
- Do not add new write handlers/commands under `internal/application/task`.
- Do not emit new `task.*` business write events.
- Typed entity writes must emit `chat.*` events only.
- Keep compatibility adapters on read/query side only.
- Keep assignee writes validated against real users.
- Do not re-introduce `TaskResult.Events`; task side effects are handled centrally in services.

Current authoritative read-model collections:
- `chats_read_model`
- `tasks_read_model`

Legacy collections (`chat_read_model`, `task_read_model`) are non-authoritative and should be cleaned for local/dev via `make reset-data`.

## Frontend & HTMX Operational Notes

- HTMX v2 WebSocket socket path is `el['htmx-internal-data'].webSocket.socket`; do not use legacy `el.__htmx_ws`.
- WS event `data` payload can contain PascalCase domain field names (for example `ChatID`) in addition to snake_case fallback handling.
- Keep HTMX close-safety guard in `web/static/js/app.js` (`htmx:wsOpen` patch) when updating websocket behavior.
- Keep first-message empty-state removal behavior in `web/templates/chat/view.html` to prevent stale placeholders.
- Echo request bind structs in handlers must include both `json:` and `form:` tags (HTMX form requests rely on `form:`).
- Task sidebar has two templates that must stay behaviorally aligned:
  - `web/templates/task/sidebar.html`
  - `web/templates/chat/task-sidebar.html`
- Chat action routes are workspace-scoped; always include `workspace_id` in path.
- JS loaded with `hx-boost` must use IIFE + global guard to avoid double initialization and redeclaration errors.

## Interface Design Rules

Follow idiomatic Go interface ownership:
- Declare interfaces on consumer side (application/domain that depends on behavior).
- Infrastructure packages implement interfaces but should not be the source of shared interface contracts.
- Keep interfaces focused and small unless consumer truly needs a larger surface.
- Prefer concrete return types and interface-typed dependencies in constructors/use cases.

## Code Style Guidelines

### Import Organization
Organize imports in three groups (separated by blank lines):
1. Standard library
2. External packages
3. Internal packages (prefixed with `github.com/lllypuk/flowra/`)

```go
import (
    "context"
    "errors"
    "fmt"
    "time"
    
    "github.com/labstack/echo/v4"
    "github.com/google/uuid"
    
    "github.com/lllypuk/flowra/internal/application/appcore"
    "github.com/lllypuk/flowra/internal/domain/chat"
)
```

### Naming Conventions
- **Files**: `snake_case.go` (e.g., `chat_handler.go`, `create_chat.go`)
- **Packages**: Short, lowercase, singular (e.g., `chat`, `user`, `httphandler`)
- **Exported**: `CamelCase` (e.g., `CreateChatUseCase`, `ChatHandler`)
- **Unexported**: `camelCase` (e.g., `chatRepo`, `validate`)
- **Interfaces**: Name by behavior, not implementation (e.g., `CommandRepository`, `EventStore`)
- **Test files**: `*_test.go` alongside implementation

### Error Handling
- **Domain errors**: Define as package-level `var` with `errors.New()`
- **Wrapping**: Use `fmt.Errorf("context: %w", err)` for wrapping
- **Validation**: Return errors early, validate at boundary layers

```go
// Package-level error constants
var (
    ErrChatNotFound = errors.New("chat not found")
    ErrNotChatMember = errors.New("not a member of this chat")
)

// Error wrapping pattern
if err := uc.validate(cmd); err != nil {
    return Result{}, fmt.Errorf("validation failed: %w", err)
}
```

### Function Signatures
- **Constructors**: `New<Type>` returns pointer and error if validation needed
- **Methods**: Use pointer receivers for aggregates/entities, value receivers for small immutable types
- **Context**: First parameter for functions that perform I/O
- **Options**: Consider functional options for complex constructors

```go
// Constructor pattern
func NewCreateChatUseCase(chatRepo CommandRepository) *CreateChatUseCase {
    return &CreateChatUseCase{chatRepo: chatRepo}
}

// Use case pattern
func (uc *CreateChatUseCase) Execute(ctx context.Context, cmd CreateChatCommand) (Result, error) {
    // Implementation
}
```

### Struct Definitions
- **Aggregates**: Unexported fields with exported getters
- **DTOs/Requests**: Exported fields with JSON/form tags
- **Constants**: Group related constants with `const` blocks

```go
// Aggregate (unexported fields)
type Chat struct {
    id           uuid.UUID
    workspaceID  uuid.UUID
    participants []Participant
    version      int
}

// DTO (exported fields with tags)
type CreateChatRequest struct {
    Name     string      `json:"name"            form:"name"`
    Type     string      `json:"type"            form:"type"`
    IsPublic bool        `json:"is_public"       form:"is_public"`
}

// Constants block
const (
    TypeDiscussion Type = "discussion"
    TypeTask       Type = "task"
    TypeBug        Type = "bug"
)
```

### Code Formatting
- **Line length**: Max 120 characters (enforced by `golines`)
- **Comments**: Full sentences with periods for exported symbols
- **Linting**: Strict golangci-lint config (see `.golangci.yml`)
  - No global variables (except package-level errors/constants)
  - No init functions
  - Exhaustive switch statements for enums
  - Shadow variable detection enabled

### Additional Style Rules
- Avoid reflection unless it is strictly necessary for dynamic external data handling.
- Prefer type assertions, type switches, interfaces, or generics over `reflect`.
- Keep code, comments, docs, error messages, and commit messages in English.

## Testing Guidelines

### Test Organization
- **Unit tests**: Beside implementation in `internal/**/*_test.go`
- **Integration tests**: `tests/integration/` with `//go:build integration`
- **E2E tests**: `tests/e2e/` with `//go:build e2e`
- **Shared utilities**: `tests/testutil/` (MongoDB/Redis/Keycloak containers)
- **Mocks**: `tests/mocks/` for shared fakes

### Test Structure (Table-Driven Pattern)
Use table-driven tests with `t.Run` for multiple scenarios:

```go
func TestCreateChat_Validation(t *testing.T) {
    tests := []struct {
        name    string
        input   CreateChatCommand
        wantErr error
    }{
        {
            name:    "valid chat",
            input:   CreateChatCommand{Name: "Test", WorkspaceID: uuid.New()},
            wantErr: nil,
        },
        {
            name:    "empty name",
            input:   CreateChatCommand{Name: "", WorkspaceID: uuid.New()},
            wantErr: ErrChatNameRequired,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Test implementation
            err := validate(tt.input)
            require.Equal(t, tt.wantErr, err)
        })
    }
}
```

### Test Naming
- **Function**: `Test<Subject>_<Scenario>` or `Test<Subject>` with grouped subtests
- **Subtests**: Use descriptive `tt.name` in table-driven tests
- **Examples**: `TestChat_NewChat`, `TestCreateChatUseCase_ValidationFailure`

### Mock Patterns
**Unit tests**: Hand-written mocks with function fields
```go
type mockChatRepo struct {
    saveFunc func(ctx context.Context, chat *chat.Chat) error
}

func (m *mockChatRepo) Save(ctx context.Context, chat *chat.Chat) error {
    if m.saveFunc != nil {
        return m.saveFunc(ctx, chat)
    }
    return nil
}
```

**Integration/E2E**: Use shared mocks from `tests/mocks/` or testcontainers

### Assertions
- Use `testify/require` for fatal checks (stops test on failure)
- Use `testify/assert` for non-fatal checks (continues test)

```go
require.NoError(t, err)              // Fatal: stop if error
assert.Equal(t, expected, actual)    // Non-fatal: continue
```

### Integration Test Setup
```go
//go:build integration

func TestChatRepository(t *testing.T) {
    // Setup real infrastructure via testcontainers
    mongoClient := testutil.SetupTestMongoDBWithClient(t)
    redis := testutil.SetupTestRedis(t)
    
    // Create isolated test environment
    repo := setupTestRepository(t, mongoClient)
    
    // Run tests
    // ...
}
```

### Running Tests
```bash
# Single test file
go test -v ./internal/domain/chat

# Single test function
go test -v ./internal/domain/chat -run TestChat_NewChat

# With build tags
go test -tags=integration -v ./tests/integration
go test -tags=e2e -v ./tests/e2e

# With coverage
go test -cover ./internal/domain/chat
```

## Commit & Pull Request Guidelines

### Commit Message Format
Use conventional commits format: `<type>: <short imperative summary>`

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code restructuring without behavior change
- `test`: Add or update tests
- `docs`: Documentation updates
- `chore`: Maintenance tasks
- `perf`: Performance improvements

**Examples**:
```
feat: add chat participant removal endpoint
fix: handle websocket reconnect timeout
refactor: simplify chat creation validation
test: add integration tests for task lifecycle
docs: update API documentation for chat endpoints
```

### Pull Request Requirements
1. **Description**: Clear summary of changes and motivation
2. **Linked issue**: Reference related issue/task number
3. **Test evidence**: Show results of `make lint` and `make test`
4. **Screenshots**: For UI changes, include before/after screenshots or GIFs
5. **Breaking changes**: Clearly document any breaking changes

### Pre-commit Checklist
- [ ] `make lint` passes (format + linter)
- [ ] `make test` passes (all tests)
- [ ] New code has test coverage
- [ ] No commented-out code or debug statements
- [ ] Documentation updated if needed

## Documentation Update Rules

- Update existing docs intentionally (`README.md`, guides, API docs) when behavior changes.
- Do not create unsolicited ad-hoc summary markdown files unless explicitly requested.
- In task-tracking markdown, do not include time estimates; track scope, dependencies, priority, and status instead.

---
> Source: [lllypuk/flowra](https://github.com/lllypuk/flowra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
