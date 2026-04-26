## github-agent-orchestrator

> This file provides context for Claude Code sessions working on gao (GitHub Agent Orchestrator).

# CLAUDE.md

This file provides context for Claude Code sessions working on gao (GitHub Agent Orchestrator).

## What is gao

A Go CLI with a Bubble Tea TUI for managing Claude Code agent sessions alongside GitHub issues and PRs. The workflow: view issues, spawn Claude Code as background processes in worktrees, monitor status via log files, track PRs, attach to sessions.

## Quick reference

```bash
make all          # lint + test + build (run before every push)
make lint         # golangci-lint (v2 config format, uses @v7 action in CI)
make test         # go test -race -count=1 ./...
go run ./cmd/gao  # run locally
```

## Project structure

```
cmd/gao/main.go           Entry point, subcommands (version, init, help)
internal/config/           YAML config at ~/.config/gao/config.yaml
internal/repo/             Auto-discover git repos under repos_dir, parse GitHub remotes
internal/github/           gh CLI wrapper (issues, PRs, browser open)
internal/process/          Background process management (start, monitor, kill, log capture)
internal/claude/           Session lifecycle, spawn, status detection
internal/statusbar/        Pluggable status bar (shell command or fallback)
internal/tui/              Bubble Tea model, view, keymap, styles
docs/                      architecture.md, configuration.md, development.md
```

See `docs/architecture.md` for full data flow, package graph, and design decisions.

## Key conventions

- **Go 1.24+**, golangci-lint v2 config (`.golangci.yml` has `version: "2"`)
- **CI**: GitHub Actions with `golangci-lint-action@v7` (not v6 - v6 only supports lint v1)
- **Error wrapping**: always `fmt.Errorf("context: %w", err)`
- **No global state**: all state flows through structs, no `init()` functions
- **External tools**: wrapped in dedicated packages under `internal/`, called via `exec.CommandContext` (not `exec.Command`), parse structured output (JSON, tab-delimited)
- **TUI pattern**: Bubble Tea `Init/Update/View` cycle. Messages are types, commands are `tea.Cmd` producers. No side effects in `View()`. Value receivers on `Init`/`Update`/`View` with `//nolint:gocritic` (required by `tea.Model` interface)
- **Pointer receivers**: use pointer receivers/params for structs > 24 bytes to satisfy gocritic's `hugeParam` check (`Config`, `Repo`, `Issue`, `Session`, `Model`)
- **File permissions**: dirs `0o750`, files `0o600` (gosec G301/G306)
- **Shell quoting**: use `shellQuote()` for values interpolated into shell commands
- **UTF-8 safety**: truncate strings by `[]rune`, not byte index

## Linter pitfalls learned in this project

These caused CI failures and are easy to hit again:

1. **golangci-lint v2 vs v1**: The `.golangci.yml` uses v2 format (`version: "2"`). CI must use `golangci-lint-action@v7`. The @v6 action only supports v1 and will fail with "unsupported version"
2. **`gofmt` and `goimports` are formatters in v2**: They go under `formatters.enable`, not `linters.enable`
3. **`gosimple` merged into `staticcheck` in v2**: Don't list it separately
4. **`typecheck` is not a linter in v2**: Remove it if present
5. **`builtinShadow` (gocritic)**: Don't name variables `max`, `min`, `new`, `len` etc. - use `lastIdx`, `maxLen`, etc.
6. **`noctx`**: All `exec.Command` calls must use `exec.CommandContext(context.Background(), ...)`
7. **`revive` exported comments**: Every exported var/const needs `// VarName ...` format
8. **`unparam`**: If a function always returns nil for one of its return values, remove that return value

## Open issues to be aware of

- **#1 - Better status detection + performance**: Includes async `RefreshStatuses()` and parallel `fetchPRs()` work
- **#3 - Pass issue body as initial prompt**: Agent starts with no context currently
- **#4 - Worktree cleanup for merged PRs**: Sessions and worktrees linger after PR merge

## Deeper context

See `.claude/` for additional reference files:
- `.claude/workflow.md` - How to work on this project effectively (PR review process, session patterns)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eulercb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
