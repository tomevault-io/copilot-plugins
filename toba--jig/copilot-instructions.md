## jig

> Multi-tool CLI combining file-based issue tracking, citation monitoring, Claude Code security guard, two-phase commit workflow, and Homebrew/Zed companion repo scaffolding.

# jig

Multi-tool CLI combining file-based issue tracking, citation monitoring, Claude Code security guard, two-phase commit workflow, and Homebrew/Zed companion repo scaffolding.

## Rules

- ALWAYS write a failing test before fixing bugs
- ALWAYS create or find an issue (`jig todo`) before starting work
- Run `scripts/lint.sh` after editing Go files

## Build & Test

```bash
go build -o jig .
go test ./...
go vet ./...
scripts/lint.sh        # golangci-lint with auto-fix, then report remaining issues
```

## Architecture

- `cmd/` — Cobra commands
  - `todo` parent with `init`, `create`, `list`, `show`, `update`, `delete`, `archive`, `roadmap`, `graphql` (alias `query`), `doctor`, `sync` (with `check`, `link`, `unlink` subcommands), `refry`, `tui` subcommands — issue tracking
  - `commit` parent with `gather`, `apply` subcommands — two-phase commit workflow
  - `cite` parent with `init`, `review` (alias `check`), `add` subcommands — citation monitoring
  - `nope` parent with `init`, `doctor`, `help` subcommands — security guard
  - `brew` parent with `init`, `doctor` subcommands — Homebrew tap management
  - `scoop` parent with `init`, `doctor` subcommands — Scoop bucket management
  - `zed` parent with `init`, `doctor` subcommands — Zed extension management
  - `prime` — output instructions for AI coding agents
  - `doctor` — run all doctor checks (nope, brew, scoop, zed)
  - `help-all` — show all commands and flags in agent-friendly format
  - `tui` — top-level alias for `todo tui`
  - `sync` — top-level alias for `todo sync` (with `check`, `link`, `unlink` subcommands)
  - `update`, `version` — top-level utilities
- `internal/config/` — `.jig.yaml` partial read/write via yaml.v3 Node API (citations section)
- `internal/cite/` — repo inspection and path suggestion for `cite add`
- `internal/github/` — GitHub API client wrapping `gh` CLI
- `internal/classify/` — Glob-based file classification (high/medium/low)
- `internal/display/` — Lipgloss-styled terminal output
- `internal/commit/` — commit workflow logic (gather/apply phases)
- `internal/companion/` — companion repo management (GitHub ops, CI workflow injection)
- `internal/nope/` — PreToolUse guard (reads `nope:` section from `.jig.yaml`)
- `internal/brew/` — Homebrew tap init and doctor logic
- `internal/scoop/` — Scoop bucket init and doctor logic
- `internal/zed/` — Zed extension init and doctor logic
- `internal/update/` — migration logic for legacy config files
- `internal/todo/config/` — todo config (reads `todo:` section from `.jig.yaml`, Node API for partial writes)
- `internal/todo/core/` — issue CRUD, archive, link checking, file watcher
- `internal/todo/graph/` — GraphQL schema and resolvers (gqlgen)
- `internal/todo/integration/` — sync integrations (ClickUp, GitHub Issues)
- `internal/todo/issue/` — issue model, frontmatter parsing, sorting
- `internal/todo/output/` — JSON output helpers
- `internal/todo/refry/` — migration from hmans/beans format
- `internal/todo/search/` — Bleve full-text search index
- `internal/todo/tui/` — Bubble Tea interactive TUI
- `internal/todo/ui/` — Lipgloss styles, tree rendering
- `pkg/client/` — GraphQL client library

## Key Design Decisions

- Config uses yaml.v3 Node API for partial read/write to avoid clobbering other sections in `.jig.yaml`
- GitHub calls shell out to `gh` CLI (no API token management needed)
- `cite review` (alias `check`) always updates `last_checked_sha`/`last_checked_date`, even when there are no new commits
- `cite add` inspects a repo via GitHub API or git clone and suggests path classification globs
- `commit` uses a two-phase gather/apply workflow so the agent can review before committing
- `brew init` and `scoop init` push to existing shared repos (`owner/homebrew-tap`, `owner/scoop-bucket`); `zed init` uses `internal/companion/` for repo creation
- `brew init` and `scoop init` auto-add to `packages` list in `.jig.yaml`
- Config uses `packages: [brew, scoop]` for convention-based repos (no URLs needed)
- Config uses `zed_extension: owner/repo` for per-project Zed extension repos
- Uses `doublestar` for `**` glob support since Go's `path.Match` lacks it
- `nope` guard reads rules from `nope:` key in `.jig.yaml` (not a separate file)
- `nope` uses instance-based `DebugLogger` (nil-safe) instead of global state
- Guard mode runs via `RunE` on the parent cobra command; exit codes use `ExitError` sentinel
- Each command group (`cite`, `nope`, `brew`, `zed`, `todo`) has its own `PersistentPreRunE`; root's is a no-op
- `nope init` writes to `.jig.yaml` and `.claude/settings.json`; migrates old `nogo`/`skill nope`/`ja nope` hooks to `jig nope`
- `todo` config uses yaml.v3 Node API for `Save()` to avoid clobbering other `.jig.yaml` sections
- `todo` stores issues as markdown files with YAML frontmatter in `.issues/`
- `todo` supports GraphQL queries/mutations via embedded gqlgen schema
- `tui` and `sync` have top-level aliases that call `initTodoCore()` in their own PreRunE

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/toba)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/toba)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
