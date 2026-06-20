## go-backend-skills

> Initialize a new Go backend repository with production-grade engineering setup. Use when creating new Go microservice projects, setting up Go repos from scratch, or when the user asks to scaffold a Go backend with best practices (Uber FX, Ent ORM, gRPC, CI/CD, linting, Docker).


# Go Backend Repository Initialization Skill

Initialize a production-grade Go backend repository following battle-tested patterns.

## When to Use

- User asks to create a new Go backend project
- User wants to scaffold a Go microservice repo
- User needs a Go project template with CI/CD, linting, Docker, etc.

## Initialization Checklist

When invoked, walk through these steps **in order**. Ask the user for project-specific details first:

1. **Project name** (Go module path, e.g., `github.com/org/my-service`)
2. **Service names** (e.g., `platform`, `api`, `worker` — at least one)
3. **Go version** (default: latest stable, currently 1.26)
4. **Database** (PostgreSQL with Ent ORM is default; ask if needed)
5. **Proto/gRPC** (yes/no — if yes, set up buf + grpc-gateway for HTTP/JSON)
6. **System dependencies** (any C libraries like libsoxr, etc.)

## Directory Structure

Create this structure, adapting service names to the user's input:

```
.
├── cmd/
│   └── <service>/
│       └── main.go              # Cobra + Uber FX app entry point
├── internal/
│   ├── configs/                 # Config loading (.env + env vars)
│   │   ├── config.go
│   │   └── module.go
│   ├── datastore/               # Database client initialization (Ent)
│   │   ├── datastore.go
│   │   └── module.go
│   ├── grpc/
│   │   ├── servers/             # gRPC + HTTP server lifecycle
│   │   ├── services/            # gRPC service implementations (business logic)
│   │   └── registers/           # Service registration with gRPC server
│   └── models/                  # Data access layer (Ent ORM CRUD)
├── pkg/                         # Reusable packages
│   ├── apierrors/               # Standardized error handling
│   └── logging/                 # slog + tint colored console logger
├── schema/                      # Ent ORM schema definitions
│   └── generate.go              # go:generate directive for Ent
├── ent/                         # (generated) Ent ORM code — DO NOT EDIT
├── api/
│   ├── proto/<service>/v1/      # .proto source files (if using gRPC)
│   └── generated/               # (generated) Proto/gRPC code — DO NOT EDIT
├── .github/
│   ├── actions/
│   │   └── setup-go-env/
│   │       └── action.yml       # Composite action: Go + system deps
│   └── workflows/
│       └── ci.yml               # Lint + Test + Build + Autofix
├── .vscode/
│   ├── settings.json
│   └── extensions.json
├── .editorconfig
├── .gitignore
├── .golangci.yml
├── .tool-versions
├── .env.example
├── buf.yaml                     # (if using gRPC) Buf CLI config
├── buf.gen.yaml                 # (if using gRPC) Proto code generation
├── cspell.config.yaml
├── renovate.json
├── Makefile                     # build, test, lint, generate, dev
├── Dockerfile
├── docker-compose.yml           # Local dev infra (postgres, etc.)
├── go.mod
└── CLAUDE.md                    # AI assistant context
```

## File Templates

### 1. `go.mod`

```go
module {{MODULE_PATH}}

go {{GO_VERSION}}

require (
	go.uber.org/fx v1.24.0
	// CLI
	github.com/spf13/cobra v1.9.1
	// Logging
	github.com/lmittmann/tint v1.1.3
	// Functional utilities
	github.com/samber/lo v1.50.0
	github.com/nekomeowww/fo v1.3.0
	github.com/nekomeowww/xo v1.9.0
	// Database
	entgo.io/ent v0.14.6
	// gRPC + gateway (if using gRPC)
	google.golang.org/grpc v1.79.3
	google.golang.org/protobuf v1.36.11
	github.com/grpc-ecosystem/grpc-gateway/v2 v2.28.0
	// Testing
	github.com/stretchr/testify v1.11.1
)
```

After creating, run `go mod tidy` to resolve dependencies.

### 2. `.editorconfig`

```ini
root = true

[*.toml]
indent_size = 4
indent_style = space
max_line_length = 100
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

[*.{js,ts,vue,tsx,jsx,html,css,json,yaml,yml}]
indent_size = 2
indent_style = space
trim_trailing_whitespace = true

[*.go]
indent_size = 2
indent_style = tab
trim_trailing_whitespace = true

[*.proto]
indent_size = 2
indent_style = space

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
```

### 3. `.golangci.yml`

See [references/golangci.yml](references/golangci.yml) for the full config.

Key principles:
- Use `version: "2"` format
- Start with `default: all`, then disable noisy/opinionated linters
- Enable `gofmt` + `goimports` formatters
- Exclude generated code paths (`api/`, `ent/`) with `generated: lax`
- Set sane thresholds: `dupl: 600`, `nestif: min-complexity: 9`

### 4. `.gitignore`

```gitignore
# Binaries for programs and plugins
*.exe
*.exe~
*.dll
*.so
*.dylib
bin/*
dist/*
out/*

# Test binary, built with `go test -c`
*.test

# Output of the go coverage tool
*.out

# Go workspace file
go.work

# Editor and IDE
.idea
*.swp
*.swo
*~
.DS_Store

# Config files (keep example)
config/*
!config/config.yaml

__debug*
.env
.env.local

# Binary outputs (add your service names)
./main
```

### 5. CI Workflow (`.github/workflows/ci.yml`)

See [references/ci.yml](references/ci.yml) for the full workflow.

Key jobs:
- **Lint** (push only): golangci-lint with 7m timeout
- **Unit Test**: `go test ./...`
- **Nilaway** (null safety): static analysis excluding generated code
- **Build Test** (PR only): `go build ./...`
- **Autofix** (PR only): auto-fix lint and commit with `[autofix]` tag, with conflict guard

### 6. Composite Action (`.github/actions/setup-go-env/action.yml`)

```yaml
name: setup-go-env
description: Setup Go environment with required dependencies
runs:
  using: composite
  steps:
    - uses: actions/setup-go@v6
      with:
        go-version: "^{{GO_VERSION}}"
        cache: true
    # Add system dependencies if needed:
    # - run: |
    #     sudo apt-get update -qq
    #     sudo apt-get install --no-install-recommends -y \
    #       libsoxr-dev \
    #       pkg-config
    #   shell: bash
```

### 7. `.vscode/settings.json`

```jsonc
{
  "go.useLanguageServer": true,
  "go.lintOnSave": "package",
  "go.vetOnSave": "workspace",
  "go.coverOnSave": false,
  "go.lintTool": "golangci-lint-v2",
  "go.lintFlags": [
    "--fast-only",
    "--config=${workspaceFolder}/.golangci.yml"
  ],
  "go.formatTool": "gofmt",
  "go.inferGopath": true,
  "go.coverOnSingleTest": true,
  "go.testTimeout": "900s",
  "go.testFlags": ["-v", "-count=1"],
  "go.toolsManagement.autoUpdate": true,
  "gopls": {
    "build.buildFlags": [],
    "ui.completion.usePlaceholders": true,
    "ui.semanticTokens": true
  },
  "[go]": {
    "editor.insertSpaces": false,
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": "always"
    }
  },
  "go.testEnvVars": {
    "LOG_LEVEL": "debug"
  }
}
```

### 8. `.vscode/extensions.json`

```json
{
  "recommendations": [
    "streetsidesoftware.code-spell-checker",
    "mikestead.dotenv",
    "EditorConfig.EditorConfig",
    "yzhang.markdown-all-in-one",
    "redhat.vscode-yaml",
    "golang.go",
    "bufbuild.vscode-buf"
  ]
}
```

### 9. `renovate.json`

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended",
    ":dependencyDashboard",
    ":semanticPrefixFixDepsChoreOthers",
    ":prHourlyLimitNone",
    ":prConcurrentLimitNone",
    ":ignoreModulesAndTests",
    "group:monorepos",
    "group:recommended",
    "group:allNonMajor",
    "replacements:all",
    "workarounds:all"
  ],
  "rangeStrategy": "bump",
  "labels": ["dependencies"],
  "minimumReleaseAge": "3 days"
}
```

### 10. `.tool-versions`

```
golang {{GO_VERSION}}
```

### 11. `cspell.config.yaml`

```yaml
version: "0.2"
ignorePaths: []
dictionaryDefinitions: []
dictionaries: []
words:
  - entgo
  - Ents
  - entsql
  - enttest
  - fxevent
  - nolint
  - stretchr
  - structpb
  - timestamppb
  # Add project-specific words as they arise
```

### 12. `Dockerfile` (multi-stage)

See [references/Dockerfile](references/Dockerfile) for the full template.

Key patterns:
- Build stage: `golang:<version>-bookworm` with system deps
- Go mod download first (layer caching)
- Build all services under `./cmd/*` dynamically
- Runtime stage: `debian:bookworm-slim` (minimal image)
- Use `--mount=type=cache` for Go build cache

### 13. Makefile

```makefile
.PHONY: build test lint generate dev

# Build binary to bin/
build:
	go build -o bin/{{SERVICE_NAME}} ./cmd/{{SERVICE_NAME}}/

# Run all tests
test:
	go test ./... -v -count=1

# Lint
lint:
	golangci-lint run ./...

# Run code generation (Ent schema + proto stubs)
generate:
	go generate ./...

# Local dev
dev:
	go run ./cmd/{{SERVICE_NAME}}/
```

### 14. Functional Utility Libraries

Always include `samber/lo`, `nekomeowww/fo`, and `nekomeowww/xo`. Prefer functional style over imperative loops.

**`samber/lo`** — Lodash-style generics utilities (slices, maps, transforms):
```go
// Filter instead of manual loops
active := lo.Filter(users, func(u *User, _ int) bool {
	return u.IsActive
})

// Map for transformations
names := lo.Map(users, func(u *User, _ int) string {
	return u.Name
})

// Find instead of range+break
admin, ok := lo.Find(users, func(u *User) bool {
	return u.Role == "admin"
})

// GroupBy for categorization
byRole := lo.GroupBy(users, func(u *User) string {
	return u.Role
})

// Coalesce for first non-zero value (config defaults)
port := lo.CoalesceOrEmpty(os.Getenv("PORT"), "8080")

// Uniq, Chunk, Flatten, Reduce, etc.
```

**`nekomeowww/fo`** — Error suppression and function invocation:
```go
// fo.May — safely unwrap (value, error), discard error
val := fo.May(strconv.Atoi(os.Getenv("PORT")))

// Common pattern: fo.May + lo.Coalesce for safe config defaults
dsn := fo.May(lo.Coalesce(os.Getenv("DATABASE_URL"), defaultDSN))

// fo.Invoke — execute with timeout
result := fo.InvokeWithTimeout(ctx, 5*time.Second, func() (string, error) {
	return fetchRemoteConfig()
})
```

**`nekomeowww/xo`** — Extended utilities (types, infra, streaming):
```go
// Type conversions, string helpers, pagination cursors, etc.
```

**Functional style guidelines:**
- **Prefer `lo.Map`/`lo.Filter`/`lo.Reduce` over manual `for` loops** for data transformations
- **Use `fo.May` for safe unwrapping** in non-critical paths (config defaults, optional values)
- **Compose functions**: chain `lo` and `fo` together for readable pipelines
- **Never use `any` type** — use generics with constraints from `lo`/`fo`/`xo`

### 15. Logging Package (`pkg/logging/`)

Use `tint` for colorized text console output. **Never use JSON handler for local dev.**

```go
package logging

import (
	"io"
	"log/slog"

	"github.com/lmittmann/tint"
)

// New constructs a colorized slog logger backed by tint.
func New(level string, w io.Writer) *slog.Logger {
	return slog.New(tint.NewHandler(w, &tint.Options{
		Level: parseLevel(level),
	}))
}

func parseLevel(level string) slog.Level {
	switch level {
	case "debug":
		return slog.LevelDebug
	case "warn":
		return slog.LevelWarn
	case "error":
		return slog.LevelError
	default:
		return slog.LevelInfo
	}
}
```

### 16. Cobra + Uber FX Entry Point (`cmd/<service>/main.go`)

```go
package main

import (
	"log/slog"
	"os"

	"github.com/spf13/cobra"
	"go.uber.org/fx"

	"{{MODULE_PATH}}/internal/configs"
	"{{MODULE_PATH}}/internal/datastore"
	"{{MODULE_PATH}}/internal/models"
	"{{MODULE_PATH}}/pkg/logging"
)

func main() {
	rootCmd := &cobra.Command{
		Use:   "{{SERVICE_NAME}}",
		Short: "{{SERVICE_DESCRIPTION}}",
		RunE: func(cmd *cobra.Command, args []string) error {
			return run()
		},
	}

	if err := rootCmd.Execute(); err != nil {
		os.Exit(1)
	}
}

func run() error {
	cfg, err := configs.Load()
	if err != nil {
		return err
	}

	logger := logging.New(cfg.LogLevel, os.Stdout)

	app := fx.New(
		// Let FX print its own lifecycle logs directly to stdout (default behavior).
		// Do NOT route FX logs through slog — it's unnecessary noise in structured logs.

		// Provide logger and config to the DI graph.
		fx.Supply(cfg),
		fx.Provide(func() *slog.Logger { return logger }),

		// Core modules
		datastore.Module(),
		models.Modules(),
		// Add service-specific modules here
	)
	app.Run()

	return nil
}
```

Key points:
- **FX uses default stdout logger** — do NOT route FX logs through slog; let FX print directly
- **`logging.New` (tint)** — colorized text output, not JSON
- **Cobra** wraps the root command for CLI extensibility (subcommands, flags)
- Logger is created outside FX, then supplied into the graph

### 17. Config Pattern (`internal/configs/`)

```go
// config.go
package configs

import (
	"os"

	"github.com/joho/godotenv"
)

type Config struct {
	LogLevel    string
	DatabaseDSN string
	HTTPPort    string
	GRPCPort    string
}

func Load() (*Config, error) {
	_ = godotenv.Load() // best-effort .env

	cfg := &Config{
		LogLevel:    getEnv("LOG_LEVEL", "info"),
		DatabaseDSN: getEnv("DATABASE_DSN", "postgres://user:pass@localhost:5432/dbname?sslmode=disable"),
		HTTPPort:    getEnv("HTTP_PORT", "8080"),
		GRPCPort:    getEnv("GRPC_PORT", "9090"),
	}

	return cfg, nil
}

func getEnv(key, fallback string) string {
	if v := os.Getenv(key); v != "" {
		return v
	}
	return fallback
}
```

```go
// module.go
package configs

import "go.uber.org/fx"

func Module() fx.Option {
	return fx.Options(
		fx.Provide(Load),
	)
}
```

### 18. FX Module Pattern

All services MUST follow this pattern — `fx.In` params struct + lifecycle hooks:

```go
package myservice

import (
	"context"
	"log/slog"

	"go.uber.org/fx"

	"{{MODULE_PATH}}/internal/configs"
	"{{MODULE_PATH}}/ent"
)

type Params struct {
	fx.In
	Config *configs.Config
	Logger *slog.Logger
	DB     *ent.Client
}

type Service struct {
	config *configs.Config
	logger *slog.Logger
	db     *ent.Client
}

func NewService(p Params) *Service {
	return &Service{
		config: p.Config,
		logger: p.Logger.With("module", "myservice"),
		db:     p.DB,
	}
}

func Module() fx.Option {
	return fx.Options(
		fx.Provide(NewService),
		fx.Invoke(func(lc fx.Lifecycle, svc *Service) {
			lc.Append(fx.Hook{
				OnStart: func(ctx context.Context) error {
					svc.logger.Info("starting myservice")
					return svc.Start(ctx)
				},
				OnStop: func(ctx context.Context) error {
					svc.logger.Info("stopping myservice")
					return svc.Stop(ctx)
				},
			})
		}),
	)
}
```

### 19. Ent ORM Schema (`schema/generate.go`)

```go
package schema

//go:generate go run -mod=mod entgo.io/ent/cmd/ent generate ./ --target ../ent --feature=sql/execquery,sql/schemaconfig,sql/lock,sql/upsert
```

Schema conventions:
- UUID primary keys: `field.UUID("id", uuid.UUID{}).Default(uuid.New).Unique().Immutable()`
- Soft delete: `field.Time("deleted_at").Optional().Nillable()`
- Enum status fields: `field.Enum("status").Values("ACTIVE", "INACTIVE")`
- Timestamps: `field.Time("created_at").Default(time.Now).Immutable()`

### 20. Datastore Module (`internal/datastore/`)

```go
package datastore

import (
	"context"
	"fmt"
	"log/slog"

	"go.uber.org/fx"

	"{{MODULE_PATH}}/ent"
	"{{MODULE_PATH}}/internal/configs"

	_ "github.com/lib/pq"
)

type Params struct {
	fx.In
	Config *configs.Config
	Logger *slog.Logger
}

func NewClient(p Params) (*ent.Client, error) {
	client, err := ent.Open("postgres", p.Config.DatabaseDSN)
	if err != nil {
		return nil, fmt.Errorf("open database: %w", err)
	}
	return client, nil
}

func Module() fx.Option {
	return fx.Options(
		fx.Provide(NewClient),
		fx.Invoke(func(lc fx.Lifecycle, client *ent.Client, logger *slog.Logger) {
			lc.Append(fx.Hook{
				OnStart: func(ctx context.Context) error {
					logger.Info("running database auto-migration")
					return client.Schema.Create(ctx)
				},
				OnStop: func(ctx context.Context) error {
					return client.Close()
				},
			})
		}),
	)
}
```

### 21. API Error Handling (`pkg/apierrors/`)

```go
package apierrors

import (
	"fmt"
	"net/http"

	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
)

type APIError struct {
	Code    int
	Message string
	Err     error
}

func (e *APIError) Error() string {
	if e.Err != nil {
		return fmt.Sprintf("%s: %v", e.Message, e.Err)
	}
	return e.Message
}

func (e *APIError) Unwrap() error { return e.Err }

func (e *APIError) GRPCStatus() *status.Status {
	return status.New(httpToGRPC(e.Code), e.Message)
}

// Sentinel errors
var (
	ErrNotFound     = &APIError{Code: http.StatusNotFound, Message: "not found"}
	ErrUnauthorized = &APIError{Code: http.StatusUnauthorized, Message: "unauthorized"}
	ErrForbidden    = &APIError{Code: http.StatusForbidden, Message: "forbidden"}
	ErrInvalidInput = &APIError{Code: http.StatusBadRequest, Message: "invalid input"}
	ErrInternal     = &APIError{Code: http.StatusInternalServerError, Message: "internal error"}
)

func httpToGRPC(code int) codes.Code {
	switch code {
	case 400:
		return codes.InvalidArgument
	case 401:
		return codes.Unauthenticated
	case 403:
		return codes.PermissionDenied
	case 404:
		return codes.NotFound
	case 409:
		return codes.AlreadyExists
	default:
		return codes.Internal
	}
}
```

### 22. Buf CLI Config (if using gRPC)

**`buf.yaml`:**
```yaml
version: v2
lint:
  use:
    - STANDARD
breaking:
  use:
    - FILE
```

**`buf.gen.yaml`:**
```yaml
version: v2
managed:
  enabled: true
  override:
    - file_option: go_package_prefix
      value: {{MODULE_PATH}}/api/generated
plugins:
  - remote: buf.build/protocolbuffers/go
    out: api/generated
    opt: paths=source_relative
  - remote: buf.build/grpc/go
    out: api/generated
    opt: paths=source_relative
  - remote: buf.build/grpc-ecosystem/gateway
    out: api/generated
    opt: paths=source_relative
  - remote: buf.build/grpc-ecosystem/openapiv2
    out: api/generated/openapiv2
    opt: merge_file_name={{SERVICE}},output_format=yaml,allow_merge=true
```

### 23. Docker Compose (local dev)

```yaml
services:
  postgres:
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: {{SERVICE_NAME}}
      POSTGRES_PASSWORD: {{SERVICE_NAME}}
      POSTGRES_DB: {{SERVICE_NAME}}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 24. `.env.example`

```bash
# =============================================================================
# Application Configuration
# =============================================================================

# Log level: debug, info, warn, error
LOG_LEVEL=info

# =============================================================================
# Database
# =============================================================================
DATABASE_DSN=postgres://{{SERVICE_NAME}}:{{SERVICE_NAME}}@localhost:5432/{{SERVICE_NAME}}?sslmode=disable

# =============================================================================
# Service Ports
# =============================================================================
HTTP_PORT=8080
GRPC_PORT=9090

# =============================================================================
# Authentication
# =============================================================================
# API_KEY=your-api-key
```

### 25. `CLAUDE.md` Template

Generate a CLAUDE.md that documents:
- Project overview (what it does, key capabilities)
- Technology stack table (layer → technology)
- Essential commands (build, test, lint, generate, dev, docker)
- Architecture (modules, layers)
- Directory structure with annotations
- DI patterns (Uber FX constructors, lifecycle hooks)
- Code style (functional-first, slog logging, error wrapping, table-driven tests)

Use the reference project's CLAUDE.md as a structural template, but tailor content to the new project.

## Post-Initialization Steps

After scaffolding, run these commands:

```bash
# Initialize Go module
go mod init {{MODULE_PATH}}
go mod tidy

# Initialize git
git init

# Generate Ent code (if using database)
go generate ./...

# Verify lint config
golangci-lint run

# Verify build
go build ./...

# Create initial commit
git add .
git commit -m "feat: initial project scaffold"
```

## Key Principles

1. **Functional-first** — prefer `lo.Map`/`lo.Filter`/`lo.Reduce` over imperative `for` loops; use `fo.May` for safe unwrapping; compose small pure functions instead of OOP inheritance
2. **Use `samber/lo`, `nekomeowww/fo`, `nekomeowww/xo`** — these are standard utilities in every project; never rewrite what they already provide
3. **Config via .env** — no YAML config files; use environment variables with `.env` for local dev
4. **Uber FX everywhere** — all DI through FX constructors with `fx.In` params structs; never manual instantiation
5. **FX default logger** — let FX use its default stdout output; do NOT wire FX logs through slog
6. **Tint for logging** — colorized text console output via `pkg/logging`; never JSON handler for dev
7. **Cobra for CLI** — wrap the root command with `cobra.Command` for flags and subcommand extensibility
8. **Generated code is sacred** — never edit `ent/` or `api/generated/`, always regenerate
9. **Soft delete by default** — `deleted_at` field on user-facing entities
10. **Structured logging** — `*slog.Logger` injected via FX with `.With("module", "name")` for context; never `fmt.Println`
11. **Error wrapping** — always `fmt.Errorf("context: %w", err)`; use `pkg/apierrors` at API boundaries
12. **Table-driven tests** — with `testify/assert`; test file suffix `_test.go` in the same package
13. **Lifecycle hooks** — use `fx.Lifecycle` with `OnStart`/`OnStop` for resources that need cleanup
14. **No `any` type** — use concrete types or generics with constraints; `any` requires explicit justification

---
> Source: [luoling8192/go-backend-skills](https://github.com/luoling8192/go-backend-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
