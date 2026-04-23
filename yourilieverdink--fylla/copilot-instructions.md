## fylla

> go build ./cmd/fylla        # Build

# Fylla - Go CLI Task Scheduler

## Build & Test Commands

```bash
go build ./cmd/fylla        # Build
go vet ./...                # Lint
go test ./...               # Test all
go test ./internal/jira/    # Test single package
go mod tidy                 # Resolve dependencies
go run ./cmd/fylla          # Run
```

## Project Structure

- Entry point: `cmd/fylla/main.go`
- Internal packages: `internal/{cli,calendar,config,github,jira,kendo,prutil,scheduler,task,timer,todoist,web}`
- Config template: `config/default_config.yaml`
- Tests: colocated `_test.go` files (Go convention)

## Architecture

### Multi-Provider System
Fylla supports multiple task providers (Jira, Todoist, GitHub, Kendo) simultaneously via the `providers` array in config. Key concepts:

- **Config:** `providers: [jira, todoist, github, kendo]` — configures which task providers to use, defaults to `["jira"]` when unset
- **Provider routing:** `isJiraKey()` / `isGitHubKey()` / `providerForKey()` infers provider from task key format (`PROJ-123` → Jira, numeric → Todoist, `repo#123` → GitHub). Kendo also uses `PROJ-123` format — disambiguation is handled via explicit `Provider` field on `task.Task` and `fylla:kendo` marker in calendar event descriptions
- **MultiTaskSource:** wraps multiple `TaskSource` instances, routes key-based operations to the correct provider. `routeToWithProvider()` uses the explicit provider name when available, falling back to key-based inference
- **multiFetcher:** concurrent fetch from all providers, merges results, handles partial failures
- **Per-provider credentials:** each provider stores credentials in a separate file (`jira_credentials.json`, `todoist_credentials.json`, `github_credentials.json`, `kendo_credentials.json`), path saved in config (`jira.credentials`, `todoist.credentials`, `github.credentials`, `kendo.credentials`)
- **Calendar descriptions:** `BuildDescriptionWithProvider()` constructs event descriptions with provider-aware markers (`fylla:` for Jira, `fylla:kendo` for Kendo, etc.) and correct URLs. `TaskKeyAndProviderFromDescription()` extracts both the task key and provider from event descriptions
- **Done marker:** `DoneMarker` (`✓ `) prefix on calendar event titles indicates completed work. `ParseTitle` strips it and sets `Done bool`. Used by `timer stop` to mark events as done.
- **Past event preservation:** `reconcile()` takes a `now` parameter and skips events whose end time is before `now`, preserving them as a calendar record
- **Worklog command:** `fylla worklog` walks calendar events (tasks + meetings), prompts for adjustments, fills remaining hours, and bulk-posts to Jira. Kendo tasks are posted directly to Kendo as time entries (bypassing the Jira fallback resolution)

### Key Interfaces
- `TaskSource` — composite interface combining all task operations (fetch, create, complete, delete, estimate, priority, due date, summary, worklog)
- `TaskFetcher` — single-method interface for fetching tasks
- `CalendarClient` — abstracts Google Calendar operations
- `Surveyor` — abstracts interactive prompts (supports `Select`, `MultiSelect`, `Input`, `InputWithDefault`, `Password`)

## Dependencies

- CLI: `github.com/spf13/cobra`
- Google Calendar: `google.golang.org/api/calendar/v3` + `golang.org/x/oauth2`
- YAML: `gopkg.in/yaml.v3`
- Interactive prompts: `github.com/AlecAivazis/survey/v2`
- GitHub API: `github.com/google/go-github/v68`
- Natural date parsing: `github.com/tj/go-naturaldate`
- HTTP, JSON, filepath, time, os: stdlib

## Code Conventions

- Follow standard Go conventions: `gofmt`, `go vet`, effective Go
- Exported types/functions get doc comments; unexported don't need them
- Errors: return `error` as last value, wrap with `fmt.Errorf("context: %w", err)`
- Naming: `MixedCaps`/`mixedCaps`, no underscores; acronyms stay caps (`URL`, `HTTP`, `JQL`)
- Interfaces: single-method interfaces named with `-er` suffix (`Reader`, `Sorter`)
- Package names: short, lowercase, singular (`config` not `configs`)
- Cobra commands: one file per command in `internal/cli/commands/`, register in `register.go`
- Config structs: use `yaml:"fieldName"` tags matching `config/default_config.yaml` keys
- Tests: table-driven with `t.Run` subtests, use `testify` only if already a dependency
- Context: pass `context.Context` as first param where needed (HTTP calls, calendar API)
- No `init()` functions; no global mutable state

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YouriLieverdink) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
