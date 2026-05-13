## wechat-mcp

> Source lives at the repository root with `main.go`, `server.go`, `routes.go`, and `tool_handlers.go` orchestrating the HTTP + MCP bridge. Domain logic is under `repository/`, grouped by feature (`chatroom`, `contact`, `message`, `tables`). Transport helpers and generated protobuf code stay in `wcferry/`. Shared utilities are in `pkg/` (SQLite setup, random IDs, file helpers). Keep new modules within this layout; add feature-specific packages beneath `repository/` and isolate cross-cutting helpers inside `pkg/`.

# Repository Guidelines

## Project Structure & Module Organization

Source lives at the repository root with `main.go`, `server.go`, `routes.go`, and `tool_handlers.go` orchestrating the HTTP + MCP bridge. Domain logic is under `repository/`, grouped by feature (`chatroom`, `contact`, `message`, `tables`). Transport helpers and generated protobuf code stay in `wcferry/`. Shared utilities are in `pkg/` (SQLite setup, random IDs, file helpers). Keep new modules within this layout; add feature-specific packages beneath `repository/` and isolate cross-cutting helpers inside `pkg/`.

## Build, Test, and Development Commands

`go mod tidy` syncs module dependencies before compiling. `go build ./...` produces a full build and validates package wiring. Run `go run main.go --sdkLibrary <path> --storageDir <data> --port 10888` for local development. Use `go test ./...` to execute all unit tests. `go fmt ./...` must run before committing to apply canonical formatting.

## Coding Style & Naming Conventions

Follow `gofmt` output (tabs for indentation, newline at EOF). Use descriptive package-level names (`repository/message`, `pkg/sqlite3`); keep exported symbols PascalCase and internal helpers camelCase. Log structured events through `logrus` or `zap` and prefer context-driven error wrapping. Filenames should remain lowercase with underscores only when mirroring external specs.

## Testing Guidelines

Author `_test.go` files beside the code they exercise using Go's `testing` package; table-driven tests are encouraged for handler and repository layers. Aim to cover new repository queries and message flows, and document edge cases using subtests. Execute `go test ./...` before pushing and attach failure output to PRs when troubleshooting.

## Commit & Pull Request Guidelines

Commits should use present-tense imperatives (`add`, `refactor`, `fix`) and stay under ~72 characters, mirroring history (`add text message sending function...`). Describe rationale and notable impacts in the body. Pull requests need: concise summary, linked issues (e.g. `Fixes #123`), manual test notes (`go run`, `go test`), and screenshots or curl transcripts for API changes. Request review once CI (if configured) is green.

## Configuration & Security Notes

Runtime flags control SDK and storage paths; keep secrets and SDK binaries outside version control (`$HOME/.wechat-mcp`). Validate new handlers against `middleware.go` to enforce authentication and sanitize data before writing through `repository/`. Never commit generated credentials or compiled SDK artifacts.

---
> Source: [lw396/wechat-mcp](https://github.com/lw396/wechat-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
