## senda

> Senda — fast, git-native API client. Collections = plain folders of YAML. Built Wails v3 (Go shell) + SolidJS (UI). Also ships TUI + headless CLI.

# CLAUDE.md

Senda — fast, git-native API client. Collections = plain folders of YAML. Built Wails v3 (Go shell) + SolidJS (UI). Also ships TUI + headless CLI.

## Architecture

- **Frontend** (`frontend/`, SolidJS + TS + CodeMirror 6): pure view + local UI state. No HTTP, no disk. Call Go via Wails bindings.
- **Backend** (Go): all network/disk/CPU work. Rule: **touch network, disk, or CPU-heavy → Go.** Frontend only render.
- **Disk = source of truth.** App stateless editor over `*.yaml` collection files.
- Two binaries: desktop GUI (`main.go` + `app*.go`, Wails/CGO/webview → `senda-desktop`) and the unified pure-Go `senda` (`cmd/senda` dispatch → TUI default + `run`/`mock`/`docs` subcommands + `gui` launcher that execs `senda-desktop`). TUI lives in `internal/tui`. The pure-Go binary has no frontend/webview, runs anywhere (servers/CI/containers).

### Go packages (`internal/`)

`httpclient` build/send requests, timing/size. `store` read/write/walk collection dirs, YAML, file watch. `model` core structs (Collection, Request, Environment, Response). `vars` resolve `{{var}}`. `auth`, `assert`, `script` (Goja JS sandbox), `runner`, `pipeline`, `history`, `importer`, `codegen`, `docgen`, `schemaval`, `security`, `mockserver`, `load`, `sseclient`, `wsclient`, `aigen`, `buildinfo`. App-bound API surface in root `app*.go`.

## Build & run

Wails v3 via Taskfile (`wails3 build` / `wails3 dev` dispatch to OS-namespaced tasks).

```
wails3 dev              # desktop, live reload
wails3 build            # prod desktop binary (stripped, ~24 MB) into bin/
task build:senda        # bin/senda (pure Go: TUI + run/mock/docs + gui launcher)
task tui -- <path>      # build + run the TUI against a collection
task generate:bindings  # regen TS bindings from Go services
```

`DEBUG=1` keep symbols (unstripped). `VERSION=x.y.z` inject version via ldflags.

## Test & check

```
go test ./...                          # Go tests
cd frontend && bun run test            # vitest
cd frontend && bun run typecheck       # tsc --noEmit
```

## Conventions

- Use `bunx`, not `npx`. Frontend deps via `bun install`.
- Frontend never do I/O — add backend work as bound Go method, regen bindings.
- Go `go 1.25.7`. Module name `senda`.
- Commit messages: no AI attribution trailers — never add `Co-Authored-By: Claude` or `Claude-Session:` lines.

---
> Source: [this-senda/senda](https://github.com/this-senda/senda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
