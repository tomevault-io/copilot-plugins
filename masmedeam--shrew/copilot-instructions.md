## shrew

> This repository is a single Go module (`go.mod`) with all runtime code in the repo root. Main entry points and core logic live in files such as `main.go`, `engine.go`, `api.go`, `server.go`, and `tui.go`. Supporting docs are in `docs/`, reusable skill prompts live in `skills/`, and static UI assets are under `ui/`. Keep tests beside the code they exercise, using `*_test.go` files in the root.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a single Go module (`go.mod`) with all runtime code in the repo root. Main entry points and core logic live in files such as `main.go`, `engine.go`, `api.go`, `server.go`, and `tui.go`. Supporting docs are in `docs/`, reusable skill prompts live in `skills/`, and static UI assets are under `ui/`. Keep tests beside the code they exercise, using `*_test.go` files in the root.

## Build, Test, and Development Commands
- `make build`: builds the local `shrew` binary for the current OS/ARCH.
- `make release-build`: cross-compiles release binaries for Linux and macOS.
- `make clean`: removes the local binary and any `shrew-*` build artifacts.
- `go test ./...`: runs the full test suite.
- `go run .`: runs the app directly during development.

## Coding Style & Naming Conventions
Use standard Go formatting only. Run `gofmt` on every edited `.go` file before finishing. Prefer short, descriptive names for package-level helpers and keep exported identifiers reserved for public behavior. Test names should follow `TestXxx` and describe one behavior, e.g. `TestSanitizeAssistantReply`.

## Testing Guidelines
The project uses Go’s built-in `testing` package. Add focused unit tests for parsing, routing, and message handling changes, and keep tests deterministic. When changing daemon, routing, or vault behavior, run `go test ./...` rather than a narrow subset. There is no separate coverage gate, but new logic should ship with a test when practical.

## Commit & Pull Request Guidelines
Recent commits use brief imperative summaries such as `Update on the readme` and `Adding bbolt database...`. Follow that style: keep commit subjects short, specific, and action-oriented. PRs should explain the change, list validation steps, and include screenshots or logs when UI, daemon, or CLI behavior changes. Link related issues when available.

## Security & Configuration Tips
Do not commit secrets, API keys, or local database snapshots. Runtime state is stored in `shrew.db`; generated files such as `shrew.db.*` and release binaries should stay out of version control. Favor the vault and environment variables for sensitive configuration.

---
> Source: [Masmedeam/shrew](https://github.com/Masmedeam/shrew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
