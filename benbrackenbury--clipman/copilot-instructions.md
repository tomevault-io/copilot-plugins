## clipman

> - **SQLite**: `github.com/mattn/go-sqlite3` (for clipboard storage)

# AGENTS.md

## Dependencies
- **SQLite**: `github.com/mattn/go-sqlite3` (for clipboard storage)
- **Clipboard**: `github.com/atotto/clipboard` (for cross-platform clipboard access)

## Build/Lint/Test Commands
- **Build**: `go build ./src/main.go`
- **Format**: `gofmt -w src/`
- **Lint**: `go vet ./src/...`
- **Test**: `go test ./src/...` (no tests currently exist)
- **Run single test**: `go test -run TestName ./src/package` (when tests are added)

## Code Style Guidelines
- **Project Structure**: Main package in `src/`, subpackages in `src/subpackage/`
- **Imports**: Standard library first, third-party second, local last (blank lines between groups)
- **Naming**: PascalCase for exported types/functions, camelCase for unexported/variables
- **Formatting**: Tabs for indentation, run `gofmt -w` before committing
- **Error Handling**: `log.Fatal()` for startup errors, `log.Printf()` for runtime errors
- **Types**: Interfaces in separate files, pointer receivers for state modification
- **Comments**: Minimal, focus on self-documenting code

---
> Source: [benbrackenbury/clipman](https://github.com/benbrackenbury/clipman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
