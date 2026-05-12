## cclog

> - `cmd/cclog`: CLI entrypoint and wiring.

# Repository Guidelines

## Project Structure & Modules
- `cmd/cclog`: CLI entrypoint and wiring.
- `internal/cli`: Flag parsing, TUI/CLI orchestration.
- `internal/parser`: JSONL parsing for Claude Code logs.
- `internal/formatter`: Filtering and Markdown generation.
- `internal/domain`: Core domain types and helpers.
- `pkg/filepicker`: Interactive TUI components (listing, preview, keybindings).
- `specs/`, `testdata/`: Product specs and sample data used by tests.

## Build, Test, and Dev Commands
- `make build`: Build the `cclog` binary from `cmd/cclog`.
- `make run`: Build then launch TUI (no args defaults to TUI).
- `make test`: Run all Go tests (`go test ./...`).
- `make test-coverage`: Run tests with coverage summary.
- `make fmt` / `make vet`: Format with `go fmt` and lint with `go vet`.
- `make deps`: Download/tidy modules. `make clean`: Remove built binaries.

Example: `go build -o cclog ./cmd/cclog/` or `go test ./internal/parser`.

## Coding Style & Naming
- Use `go fmt` (run `make fmt`) and keep `go vet` clean.
- Idiomatic Go: exported identifiers in `CamelCase`, packages lowercase, short, no underscores.
- Keep functions small and table-driven for parsing/formatting logic.
- File naming: Go files `*.go`; tests `*_test.go` next to the code.

## Testing Guidelines
- Framework: standard library `testing`. Prefer table-driven tests.
- Location: co-locate tests with packages (e.g., `internal/formatter/markdown_test.go`).
- Run: `make test` for full suite; add focused runs like `go test ./internal/cli -v`.
- Coverage: use `make test-coverage`; avoid regressing coverage on changed packages.
- Use fixtures from `testdata/` where realistic logs are required.

## Commit & Pull Request Guidelines
- Commit style: Conventional Commits (`feat:`, `fix:`, `refactor:`) with concise scope, e.g., `feat(tui): add search help`.
- PRs must include: clear rationale, what/why, linked issues, and TUI screenshots/GIFs when UI changes.
- Require: passing `make fmt vet test`, updated docs/specs if behavior changes.

## Security & Configuration Tips
- Do not commit real logs or secrets; sample data belongs in `testdata/`.
- Respect user environment: default TUI root is `~/.claude/projects` when unset; opening files uses `$EDITOR`.
- `.gitignore` already excludes build artifacts and `.serena/cache/`—keep it that way.

---
> Source: [annenpolka/cclog](https://github.com/annenpolka/cclog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
