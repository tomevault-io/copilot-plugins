## canvas-cli

> This file provides guidance to AI agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

# AGENTS.md

This file provides guidance to AI agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Build & Development Commands

```bash
# Build
make build              # Build binary to bin/canvas
make dev                # Build with fmt and vet

# Test
make test               # Run all tests
make test-coverage      # Run tests with coverage report
go test -v ./internal/api/...  # Run specific package tests
go test -v -run TestName ./... # Run single test

# Lint & Format
make fmt                # Format code
make lint               # Run golangci-lint
make vet                # Run go vet

# Install
make install            # Install to /usr/local/bin
make uninstall          # Remove from /usr/local/bin

# Setup
make setup-hooks        # Install git pre-commit hooks
```

## Pre-commit Hook

Run `make setup-hooks` to enable. Runs automatically on each commit:
- `gofmt` - formatting check
- `golangci-lint` - comprehensive linting (if installed)
- `go vet` - static analysis
- `go test -short -race` - quick test pass with race detector

## Architecture

Canvas CLI is a Go CLI for Canvas LMS API, built with Cobra/Viper.

### Project Structure

```
cmd/canvas/     → Entry point (main.go)
commands/       → Cobra command definitions (one file per resource)
  internal/
    options/    → Option structs for commands (eliminates global state)
    logging/    → Structured logging for commands
internal/
  api/          → Canvas API client + service layer (Client, *Service structs)
  auth/         → OAuth 2.0 + PKCE, token storage (keyring/encrypted file)
  config/       → Viper-based configuration management
  cache/        → Response caching with TTL
  batch/        → Concurrent batch operations (worker pool)
  diagnostics/  → canvas doctor checks
  dryrun/       → --dry-run curl rendering
  output/       → Formatters (table, JSON, YAML, CSV)
  progress/     → Progress indicators
  repl/         → Interactive shell
  shellparse/   → Shell-style argument parsing
  telemetry/    → Opt-in usage telemetry
  terminal/     → Terminal capabilities
  update/       → Self-update checks
  webhook/      → Webhook listener
testdata/spec/  → Committed Canvas API spec manifest
tools/          → Code generators (gendocs, speccheck)
.ai/            → Canvas LMS API documentation (gitignored)
```

### Key Patterns

**Service Layer**: Each Canvas resource has a service in `internal/api/`:
```go
type ModulesService struct { client *Client }
func NewModulesService(client *Client) *ModulesService
```

**Command Pattern (NEW)**: Commands should use options structs instead of global flags:
```go
// commands/internal/options/resource.go
type ResourceListOptions struct {
    CourseID int64
    Include  []string
}

func (o *ResourceListOptions) Validate() error {
    return ValidateRequired("course-id", o.CourseID)
}

// commands/resource.go
func newResourceListCmd() *cobra.Command {
    opts := &options.ResourceListOptions{}
    cmd := &cobra.Command{
        Use: "list",
        RunE: func(cmd *cobra.Command, args []string) error {
            if err := opts.Validate(); err != nil {
                return err
            }
            client, _ := getAPIClient()
            return runResourceList(cmd.Context(), client, opts)
        },
    }
    cmd.Flags().Int64Var(&opts.CourseID, "course-id", 0, "Course ID")
    return cmd
}
```

**Structured Logging (NEW)**: Commands should use structured logging:
```go
import "github.com/jjuanrivvera/canvas-cli/commands/internal/logging"

func runCommand(ctx context.Context, client *api.Client, opts *Options) error {
    logger := logging.NewCommandLogger(globalDebugFlag)

    logger.LogCommandStart(ctx, "resource.list", map[string]interface{}{
        "course_id": opts.CourseID,
    })

    // ... perform operation ...

    logger.LogCommandComplete(ctx, "resource.list", len(results))
    return nil
}
```

**API Client**: `internal/api/client.go` provides `HTTPClient` interface with:
- Automatic pagination (`GetAllPages`)
- Adaptive rate limiting based on Canvas quota headers
- Exponential backoff retry

### Testing

Tests use `httptest.NewServer` for mock HTTP servers. Service tests follow pattern:
```go
func TestServiceMethod(t *testing.T) {
    server := httptest.NewServer(...)
    client := &Client{BaseURL: server.URL, ...}
    service := NewXxxService(client)
    // test service methods
}
```

Use `t.Fatal()` (not `t.Error()`) when nil checks would cause subsequent panics.

## Branching & Release

### Branch Model (Simplified Git Flow)

```
main     ──●─────────────────●──────► (tagged releases)
           │                 ↑↓
develop  ──●───●───●───●─────●──────► (integration)
               ↑       ↑
feature/*  ────┘       │
fix/*  ────────────────┘
```

| Branch | Purpose | Merges To |
|--------|---------|-----------|
| `main` | Tagged releases only | - |
| `develop` | Integration (PR target) | `main` on release |
| `feature/*` | New features | `develop` |
| `fix/*` | Bug fixes | `develop` |
| `hotfix/*` | Urgent fixes | `main` AND `develop` |

### When develop syncs with main

1. **After a release**: Merge `main` back to `develop` to capture release commits
2. **After a hotfix**: Hotfix merges to both `main` and `develop`

### Release Process

Before tagging, on `develop`:

1. **Update CHANGELOG.md** with the new version section (and copy to
   `docs/changelog.md` — `make docs-gen` does this automatically)
2. **Update SECURITY.md** supported-versions table if the minor version changes

Then:

```bash
# 1. Merge develop to main
git checkout main && git merge develop

# 2. Tag and push
git tag -a v1.x.x -m "Release v1.x.x"
git push origin main --tags

# 3. Sync main back to develop
git checkout develop && git merge main
git push origin develop
```

GitHub Actions automatically builds binaries and creates the release on tag push.

## CI

Single workflow `.github/workflows/ci.yml` runs:
- Lint (gofmt, go vet, golangci-lint)
- Security (govulncheck and gosec — both blocking)
- Test matrix (ubuntu/macos/windows, Go version from go.mod) with a **total
  coverage gate ≥80%** on ubuntu (`go tool cover -func` over `./...`)
- Binary-level integration tests (`-tags integration`, ubuntu)
- Build artifacts (GoReleaser snapshot)

Run everything CI runs locally with `make check`.

## API Spec Compliance & Coverage

The CLI is validated against Canvas's **official API spec** (Swagger 1.2),
committed at `testdata/spec/canvas_endpoints.json` (1086 endpoints) with
response models at `testdata/spec/canvas_models.json`.

- `internal/api/spec_contract_test.go` is a network-free test (runs in the
  normal suite) that harvests every `/api/v1/...` path the service layer calls
  and asserts each matches a documented Canvas endpoint. **A new endpoint with a
  wrong path fails the build** — this is the regression guard that has already
  caught several real path bugs.
- `make spec-sync` regenerates the manifest by fetching the official Swagger
  from a live Canvas host (`-host`/`CANVAS_SPEC_HOST`, default
  `learn.canvas.net`; canvas.instructure.com IP-blocks datacenter requests).
  Do NOT use the gitignored `.ai/canvas-lms-docs` mirror as the source — the
  committed manifest is authoritative.
- `make spec-coverage` prints documented-but-unimplemented endpoints (the
  coverage gap), grouped by resource.

When adding endpoints to maximize coverage:
1. Take exact paths/verbs from the committed manifest (and field names from
   `canvas_models.json`) — the contract test enforces the path.
2. **Add proportional tests.** Every new command needs cmdtest coverage
   (run function + options `Validate()`) or the 80% gate drops. The `commands`
   and `commands/internal/options` packages are where coverage erodes fastest.
3. If parallelizing across resources, **partition strictly by file** and
   **declare each shared model type in exactly one file**. Multiple agents
   independently declaring the same struct (e.g. `Feature`, `ContentExport`,
   `GradingPeriod`) or the same dual-scoped resource (account vs course
   `grading_standards`) causes redeclaration/merge collisions — the main
   integration cost of fan-out work.

## Documentation

Documentation is built with MkDocs Material and deployed to GitHub Pages.

**Live site**: https://jjuanrivvera.github.io/canvas-cli/

### Local Development

```bash
# Install dependencies
pip install mkdocs-material mkdocs-git-revision-date-localized-plugin

# Generate CLI reference and serve locally
go run ./tools/gendocs/main.go
mkdocs serve
```

### Deployment

Documentation auto-deploys on push to `main` when `docs/**` or `mkdocs.yml` changes.

**Manual trigger** (via GitHub UI):
1. Go to Actions → Documentation workflow
2. Click "Run workflow"

**Manual trigger** (via CLI):
```bash
gh workflow run docs.yml
```

**If deployment gets stuck** in "queued" status:
```bash
# Force a Pages build
gh api -X POST repos/jjuanrivvera/canvas-cli/pages/builds

# Check status
gh api repos/jjuanrivvera/canvas-cli/pages --jq '.status'
```

## Releases

Releases use GoReleaser and auto-publish to GitHub Releases + Homebrew tap.

### Creating a Release

Pre-tag checklist (on `develop`):

1. Update `CHANGELOG.md` with the new version section, then run `make docs-gen`
   to sync `docs/changelog.md`
2. Update the supported-versions table in `SECURITY.md` if the minor version
   changes

```bash
# 1. Ensure main is up to date
git checkout main && git merge develop

# 2. Create and push tag
git tag -a v1.x.x -m "Release v1.x.x"
git push origin main --tags

# 3. Sync develop
git checkout develop && git merge main
git push origin develop
```

GoReleaser automatically:
- Builds binaries for linux/darwin/windows (amd64/arm64)
- Creates GitHub release with changelog
- Updates Homebrew formula in `jjuanrivvera/homebrew-canvas-cli`

### Homebrew Tap

The formula is at: https://github.com/jjuanrivvera/homebrew-canvas-cli

**Required secret**: `HOMEBREW_TAP_TOKEN` - a PAT with `repo` scope for the tap repository

## Technical Debt & Remediation

See [TECHNICAL_DEBT.md](TECHNICAL_DEBT.md) for tracked technical debt items. The migration away from package-level flag variables is complete; the remaining globals in `commands/root.go` are an accepted design choice documented there.

### Adding New Commands

When adding new commands, follow these patterns:

1. **Create option struct** in `commands/internal/options/`
2. **Use structured logging** from `commands/internal/logging`
3. **Avoid global flag variables**
4. **Add tests** for command logic
5. **Validate options** before execution

Example:
```go
// 1. Define options
type NewResourceOptions struct {
    RequiredField string
    OptionalField int
}

func (o *NewResourceOptions) Validate() error {
    return ValidateRequired("required-field", o.RequiredField)
}

// 2. Create command with logging
func newNewResourceCmd() *cobra.Command {
    opts := &options.NewResourceOptions{}
    cmd := &cobra.Command{
        Use: "new-resource",
        RunE: func(cmd *cobra.Command, args []string) error {
            logger := logging.NewCommandLogger(globalDebugFlag)

            if err := opts.Validate(); err != nil {
                return err
            }

            logger.LogCommandStart(cmd.Context(), "new.resource", map[string]interface{}{
                "required_field": opts.RequiredField,
            })

            // Implementation

            logger.LogCommandComplete(cmd.Context(), "new.resource", recordCount)
            return nil
        },
    }
    cmd.Flags().StringVar(&opts.RequiredField, "required-field", "", "Required field")
    return cmd
}
```

---
> Source: [jjuanrivvera/canvas-cli](https://github.com/jjuanrivvera/canvas-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
