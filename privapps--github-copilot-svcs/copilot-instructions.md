## github-copilot-svcs

> Source code location:

# Repository Guidelines

## Project Structure & Module Organization

Source code location:
- `cmd/` — Application entry points
- `internal/` — Core service modules (auth, config, API, middleware)
- `pkg/` — Shared utilities/packages
- `test/` — Integration and helper tests
- `config.example.json`, `Dockerfile`, `docker-compose.yml` — Example/config files

## Build, Test, and Development Commands

Key commands (via Makefile):
- `make build` — Build service binary
- `make run` — Start proxy server locally
- `make dev` — Hot reload (requires air)
- `make test` — Unit tests
- `make test-all` — All tests (unit + integration)
- `make test-coverage` — Coverage report
- `make lint` — Lint code (golangci-lint)
- `make fmt` — Format code

Requires Go 1.23.0+

## Coding Style & Naming Conventions

- Indentation: tabs (Go standard)
- Use camelCase or snake_case for names
- Exported Go identifiers: PascalCase
- Format code before PRs (`make fmt`), lint (`make lint`)

## Testing Guidelines

- Use Go `testing` package; name test files `_test.go`, test functions `TestXxx`
- Unit tests: `internal/` and `pkg/`
- Integration tests: `test/integration/`
- Run: `make test-all`, `make test-coverage` (aim for >=45% coverage in core logic)

## Commit & Pull Request Guidelines

- Commit messages: short, present-tense (e.g., "Refactor code structure")
- PRs: describe changes/reasoning, link issues, add screenshots for UI
- Ensure all tests pass & code is formatted
- Do not commit secrets or sensitive configs

## Security & Configuration Tips

- Store secrets in user-level config with permissions 0700
- Never log sensitive data
- Only use HTTPS for credentials/tokens
- Do not push sensitive files; check `.gitignore`

---

For help, open an issue or see the README troubleshooting section.

---
> Source: [privapps/github-copilot-svcs](https://github.com/privapps/github-copilot-svcs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
