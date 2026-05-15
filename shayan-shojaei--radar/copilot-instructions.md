## radar

> radar is a terminal-based API explorer. It accepts a Swagger or OpenAPI spec URL,

# radar — CLAUDE.md

## What radar does

radar is a terminal-based API explorer. It accepts a Swagger or OpenAPI spec URL,
parses it, and launches a Bubble Tea TUI. Users browse endpoints, fill in params
and body fields, fire HTTP requests, and inspect responses. All session data
(request bodies, headers, cookies) is saved as age-encrypted JSON files.

User flow:
1. `radar --url <spec>` → fetch + parse spec → launch TUI
2. Endpoint list view → filter / select an endpoint
3. Request editor → fill in base URL, path params, query params, headers, cookies, body
4. `Ctrl+S` to send → response viewer shows status, headers, pretty-printed body
5. Session auto-saved; `Ctrl+L` reloads a previous request for the same endpoint

## Commands

```sh
make run url=https://...   # run in development
make build                 # produce bin/radar
make test                  # run all tests
make lint                  # golangci-lint run
make release               # cross-compile linux/amd64, darwin/amd64, darwin/arm64
```

## Architecture

```
cmd/radar/        CLI wiring only (cobra). No business logic.
internal/
  tui/            Root Bubble Tea model (app.go) + message types (messages.go)
  tui/views/      Sub-models: list.go, request.go, response.go
  tui/components/ Reusable TUI components (input fields, panels)
  openapi/        Fetch + parse spec → []models.ParsedEndpoint
  requester/      Build + fire HTTP requests → models.ResponseData
  session/        Encrypted persistence of RequestData per endpoint
  crypto/         Thin wrapper around filippo.io/age (Encrypt/Decrypt only)
  config/         Read env vars with defaults (RADAR_STORAGE_DIR, RADAR_TIMEOUT)
pkg/
  models/         Shared structs: ParsedEndpoint, RequestData, ResponseData, Session
```

## Bubble Tea Patterns

- One top-level `Model` in `tui/app.go` holds `state viewState` and owns sub-models
- Each view (`ListModel`, `RequestModel`, `ResponseModel`) has its own `Update`/`View`
- Views communicate with the root via Msg types in `tui/messages.go`
- All IO (HTTP requests, file reads) happens inside `tea.Cmd` closures — never block `Update`
- `tea.WindowSizeMsg` is forwarded to all sub-models regardless of active state

## Go Conventions

- Errors: always wrap with `fmt.Errorf("context: %w", err)`
- No global state — inject dependencies via constructors
- Interfaces defined where they are consumed, not where implemented
- Table-driven tests with `t.Run()`
- Config only via environment variables, never hardcoded values

## Encryption Rules

- All session data is encrypted with age before writing to disk
- Passphrase is prompted at startup if a session file exists; set `RADAR_PASSPHRASE` to skip
- Never log or print decrypted session content
- Never store the passphrase in memory longer than necessary

## What NOT to do

- Do not use a web framework
- Do not use GORM or any ORM
- Do not put HTTP logic inside TUI views — use `tea.Cmd` to delegate
- Do not block inside Bubble Tea's `Update` function
- Do not add a framework unless explicitly asked

---
> Source: [shayan-shojaei/radar](https://github.com/shayan-shojaei/radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
