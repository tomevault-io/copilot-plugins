## things3-cli

> - `cmd/things/` is the CLI entry point.

# Repository Guidelines

## Project Structure & Module Organization
- `cmd/things/` is the CLI entry point.
- `internal/cli/` defines commands, help text, and output formatting.
- `internal/db/` contains Things database models, queries, and read-only access.
- `internal/things/` implements URL scheme encoding and add/update logic.
- `internal/osascript/` wraps AppleScript helpers for area/project automation.
- `integration/` holds integration tests and fixtures; unit tests live alongside code in `internal/**`.
- `docs/`, `doc/`, and `share/` include documentation and man page sources; `Formula/` contains the Homebrew formula.
- `scripts/` contains release automation helpers.

## Build, Test, and Development Commands
- `make build` — builds the CLI binary into `bin/things`.
- `make test` — runs all unit and integration tests (`go test ./...`).
- `make install` — builds and installs the binary (and man page if present) to `/usr/local` or `PREFIX`.
- `make uninstall` — removes installed binary and man page.
- Example direct Go commands: `go test ./integration`, `go build -o bin/things ./cmd/things`.

## Coding Style & Naming Conventions
- Go code follows standard `gofmt` formatting (tabs for indentation).
- Package names are lower-case, short, and domain-focused (`cli`, `db`, `things`).
- Command files in `internal/cli/` mirror subcommand names (e.g., `add.go`, `update.go`).
- Tests use the Go convention `*_test.go`.

## Testing Guidelines
- Tests use Go’s built-in `testing` package.
- Unit tests live next to their packages (e.g., `internal/db/*_test.go`).
- Integration tests live in `integration/` and include sqlite fixtures and helpers.
- Run the full suite with `make test` or target a package with `go test ./internal/cli -run TestName`.

## Commit & Pull Request Guidelines
- Recent commits use short, imperative summaries; common prefixes include `feat:`, `chore:`, and `docs:`.
- If you use a prefix, keep it concise and match the scope of the change.
- No PR template is defined; include a brief summary and testing notes.
- Call out macOS/Things-specific behavior changes and any permissions needed.

## Configuration & Permissions Notes
- Update commands require `THINGS_AUTH_TOKEN`; see README for setup.
- DB reads may require Full Disk Access for your terminal.
- Area/project automation uses AppleScript and may trigger macOS prompts.

## Agent Scripts / Skills Sync
- Keep the in-repo skill mirror at `skills/things/SKILL.md` in sync with
  `../agent-scripts/skills/things/SKILL.md`.
- After CLI changes, update both copies so agent guidance stays aligned.

---
> Source: [ossianhempel/things3-cli](https://github.com/ossianhempel/things3-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
