## ordercli

> - `cmd/ordercli/`: CLI entrypoint (`main.go`) and top-level wiring.

# Repository Guidelines

## Project Structure & Module Organization
- `cmd/ordercli/`: CLI entrypoint (`main.go`) and top-level wiring.
- `internal/cli/`: Cobra commands, CLI state, helpers, and provider routing.
- `internal/foodora/` and `internal/deliveroo/`: provider clients, models, and behavior.
- `internal/config/`: config loading/validation and JWT helpers.
- `internal/browserauth/` + `internal/chromecookies/`: Playwright/Chrome cookie/session helpers.
- Tests live alongside code as `*_test.go` in the same directories.

## Build, Test, and Development Commands
- `go build ./cmd/ordercli`: build the CLI binary.
- `go test ./...`: run the full test suite.
- `make test`: same as `go test ./...`.
- `make fmt`: format with `gofumpt`.
- `make lint`: run `golangci-lint`.
- `make tools`: install `gofumpt` and `golangci-lint`.

## Coding Style & Naming Conventions
- Go formatting via `gofumpt` (run `make fmt`).
- Lint via `golangci-lint` (run `make lint`).
- Follow standard Go naming: exported identifiers in `CamelCase`, unexported in `camelCase`.
- Tests use Go’s `*_test.go` naming and `TestXxx` functions.

## Testing Guidelines
- Primary framework: Go’s `testing` package.
- Run all tests with `go test ./...` before shipping changes.
- Add focused unit tests next to the code being changed (e.g., `internal/foodora/client_test.go`).

## Commit & Pull Request Guidelines
- Commit messages follow Conventional Commit style (`docs: …`, `test: …`, `fix: …`).
- Keep commits scoped and descriptive; prefer one logical change per commit.
- PRs should include a short summary, key commands run (e.g., `go test ./...`), and any relevant usage examples.

## Configuration & Runtime Notes
- Config defaults to the OS config directory; override for testing with:
  - `./ordercli --config /tmp/ordercli.json foodora config show`
- Provider-specific auth flows are documented in `README.md` (Foodora/Deliveroo sections).

## Food Requests (Agent Behavior)
- When Peter asks for food/foodora orders, return full order details by default (not summary-only).

---
> Source: [steipete/ordercli](https://github.com/steipete/ordercli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
