## spec

> This repository is a Go workspace for `github.com/oaswrap/spec`. Core package files live at the repository root and in `openapi/`, `option/`, `pkg/`, and `internal/`. Router adapters are separate modules under `adapter/<framework>openapi/`, each with its own examples, tests, and `go.mod`. Runnable examples live in `examples/` and adapter `example/` directories. Golden YAML fixtures and expected OpenAPI output live in `testdata/` and adapter `testdata/` directories.

# Repository Guidelines

## Project Structure & Module Organization

This repository is a Go workspace for `github.com/oaswrap/spec`. Core package files live at the repository root and in `openapi/`, `option/`, `pkg/`, and `internal/`. Router adapters are separate modules under `adapter/<framework>openapi/`, each with its own examples, tests, and `go.mod`. Runnable examples live in `examples/` and adapter `example/` directories. Golden YAML fixtures and expected OpenAPI output live in `testdata/` and adapter `testdata/` directories.

## Build, Test, and Development Commands

- `make install-tools`: installs `golangci-lint`.
- `make test`: runs core and adapter tests with `go test`.
- `make test-adapter`: runs only adapter module tests.
- `make test-update`: regenerates golden files when output changes intentionally.
- `make testcov` / `make testcov-html`: produces coverage reports in `coverage/`.
- `make lint`: runs `golangci-lint` for core and adapters.
- `make tidy`: runs `go mod tidy` across core, adapters, and examples.
- `make sync`: updates the Go workspace.
- `make check`: runs sync, tidy, lint, and all tests.

Use `go test ./... -run TestName` for focused core tests, or run the same command inside an adapter module for adapter-specific work.

## Coding Style & Naming Conventions

Use standard Go formatting (`gofmt`) and idiomatic Go naming. Package names are short lowercase names such as `openapi`, `option`, or `parser`. Test files must use the `_test.go` suffix. Adapter directories follow the `<framework>openapi` pattern, for example `fiberopenapi` and `ginopenapi`. Keep public APIs documented when exported names are not self-explanatory.

## Testing Guidelines

Tests use Go's standard `testing` package, with golden-file coverage for generated OpenAPI YAML. Add or update tests for behavior changes, especially changes to route registration, schema reflection, options, or adapter output. When running `make test-update`, review all generated fixture diffs before committing.

## Commit & Pull Request Guidelines

Use Conventional Commits:

```text
feat: add response header option
fix: handle empty path group correctly
docs: clarify WithSecurity usage
```

Accepted types include `feat`, `fix`, `docs`, `style`, `refactor`, `test`, and `chore`. Keep commits focused. Pull requests should describe what changed, why it changed, affected modules or adapters, related issues, and tests run. Include fixture updates when generated output changes.

## Agent-Specific Instructions

Follow existing patterns before adding abstractions. Changes to core generation may require adapter and golden-file updates. Do not modify release tags, generated coverage reports, or unrelated module files unless the task explicitly requires it.

---
> Source: [oaswrap/spec](https://github.com/oaswrap/spec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
