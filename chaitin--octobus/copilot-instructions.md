## octobus

> - OctoBus is a Go module that builds a single `octobus` binary.

# Repository Guidelines

## Project Structure & Module Organization

- OctoBus is a Go module that builds a single `octobus` binary.
- The CLI entry point lives in `cmd/octobus`.
- Core implementation is split by concern under `internal/`: `admin`, `cli`, `descriptors`, `domain`, `packageimport`, `protocol`, `server`, `store`, and `supervisor`.
- Integration tests sit in `internal/integration`.
- Full daemon and CLI scenarios live in `tests/e2e`.
- Design notes are in `docs/design`.
- The JavaScript service fixture is in `examples/calculator-js`.

## Build, Test, and Development Commands

Use Task as the main workflow:

```bash
task          # run lint, test, and build
task lint     # verify gofmt and run go vet ./...
task test     # run go test ./...
task build    # build bin/octobus from ./cmd/octobus
```

- For focused work, run package tests directly, for example `go test ./internal/store`.
- Run e2e tests with `go test ./tests/e2e -count=1`; these build the binary, start a daemon, and exercise admin, gRPC, REST, MCP, and reflection paths.
- E2e tests and service imports require Node.js/npm and `protoc`.

## Coding Style & Naming Conventions

General rules:

- Keep Go code `gofmt` formatted and `go vet` clean.
- Use tabs via `gofmt`, short package names, and explicit errors with useful context.
- Prefer standard library facilities when they fit the problem.
- Use well-established open source libraries for foundational concerns outside the standard library instead of hand-rolling basic plumbing.
- Avoid building custom infrastructure without a clear project-specific need.

Project-specific rules:

- Follow existing domain terms: service, instance, capset, method binding, descriptor, runtime, and artifact.
- Test files should use `*_test.go` naming.
- The JavaScript example is an ES module package; keep scripts under `examples/calculator-js/bin` and proto files under `examples/calculator-js/proto`.

## Testing Guidelines

- Place unit tests beside the package they cover, such as `internal/domain/domain_test.go`.
- Put cross-component flows in `internal/integration`.
- Put process-level scenarios in `tests/e2e`.
- Unit tests, integration tests, and e2e tests must each have at least 60% coverage, based on `task test` output.
- Overall test coverage must be at least 90%, based on `task test` output.
- Prefer deterministic tests with temporary data dirs, not a user’s `~/.octobus`.
- Before opening a PR, run `task`.
- Run e2e tests after changes to daemon startup, CLI commands, package import, routing protocols, or supervision.

## Commit & Pull Request Guidelines

Commit rules:

- Use short imperative summaries, sometimes with a scope prefix, as in `Add calculator JS example service`, `docs: update Chinese README usage guide`, or `implement service instance capset CRUD commands`.
- Keep commits focused.
- Describe changed behavior, not just touched files.

Pull request rules:

- Include a concise summary.
- Include tests run.
- Link issues when applicable.
- Note changes that affect CLI behavior, storage schema, service package format, or local runtime requirements.
- Include screenshots only for documentation or UI changes.

## Security & Configuration Tips

- Do not commit `bin/` output, local data dirs, logs, packaged service artifacts, or instance config secrets.

---
> Source: [chaitin/OctoBus](https://github.com/chaitin/OctoBus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
