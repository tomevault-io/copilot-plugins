## debug-skill

> **dap-cli** (`dap`) is a CLI debugging tool for AI agents. It wraps the Debug Adapter Protocol (DAP) behind a simple CLI that agents call via Bash. A background daemon holds the stateful debug session; the CLI sends ephemeral commands via Unix socket.

# CLAUDE.md

## Project Overview

**dap-cli** (`dap`) is a CLI debugging tool for AI agents. It wraps the Debug Adapter Protocol (DAP) behind a simple CLI that agents call via Bash. A background daemon holds the stateful debug session; the CLI sends ephemeral commands via Unix socket.

**Primary users**: AI coding agents (Claude Code, Cursor, etc.)
**Supported languages**: Python (debugpy), Go (dlv), Node.js/TypeScript (js-debug), Rust/C/C++ (lldb-dap) — all via DAP backends

Remember to format the code with `go fmt ./...`

## Quick Reference

```bash
make build      # Build binary
make test       # Run all tests
make lint       # Run go vet + staticcheck
make all        # lint + test + build
```

## Architecture

```
CLI (cobra) → Unix socket → Daemon (holds DAP session) → TCP/stdio → Debug Adapter → Target
```

## Design Docs

Read `claudedocs/` for full context — these are the source of truth for architecture, API, and implementation plan:
- `claudedocs/design.md` — Architecture, decisions, inspirations
- `claudedocs/api.md` — Complete CLI API reference
- `claudedocs/implementation.md` — Implementation plan with phases


## Code Conventions

- **Go 1.22+**
- **Formatting**: `gofmt` (standard)
- **Linting**: `go vet` + `staticcheck`
- **Testing**: TDD — write test first, then implement
- **Error handling**: Return errors, don't panic. Wrap with context via `fmt.Errorf("doing X: %w", err)`
- **Naming**: Go conventions (exported = PascalCase, unexported = camelCase)
- **Dependencies**: Minimal. Only google/go-dap and cobra. Stdlib for everything else.
- **Principles**: KISS, DRY, flat file structure. Short and effective codebase. Every line earns its place.

## Key Design Patterns

- **Auto-Context**: Every execution command (`debug`, `step`, `continue`) blocks until stopped, then returns full context (location + source + locals + stack + output). No follow-up calls needed.
- **Invisible Daemon**: Background process is an implementation detail. `dap debug` auto-starts it; `dap stop` kills it. No user-facing daemon management.
- **Backend Abstraction**: Pluggable debug adapters (debugpy, dlv, js-debug, lldb-dap) behind a common interface. See `claudedocs/design.md`.
- **Multi-Session**: `--session <name>` maps to independent daemon + socket (`~/.dap-cli/<name>.sock`). Default session = `"default"`. Idle timeout (10 min) prevents orphan daemons.
- **Length-prefixed JSON IPC**: CLI ↔ daemon communication over Unix socket. One request, one response per connection.

## Testing

```bash
go test ./...                    # All tests
go test -run TestName            # Specific test
go test -v                       # Verbose
```

- Unit tests: mock DAP client via interface
- Integration tests: real debug adapter processes
- E2e tests: build binary, run full CLI flow

---
> Source: [AlmogBaku/debug-skill](https://github.com/AlmogBaku/debug-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
