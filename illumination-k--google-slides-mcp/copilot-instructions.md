## google-slides-mcp

> This repository is designed to be automation-friendly.

# AGENTS.md

This repository is designed to be automation-friendly.
Human contributors and AI agents should follow this document to keep changes small, reproducible, and consistently formatted.

## Golden Rule: Code Quality is enforced via mise

Tooling and quality gates are pinned and executed through **mise**.
Do **not** rely on globally installed `go`, `golangci-lint`, or `dprint`.

- Source of truth: `.mise.toml`
- Tool version lock: `mise.lock` (`[settings].locked = true` is enabled)

This means:

- Running `mise install` (or `mise trust` when prompted) yields the same tool versions for everyone.
- CI and local runs should match because tool versions are locked.
- If you update tool versions, you must update both `.mise.toml` and `mise.lock`.

### Required quality commands

Run these from the repository root.

- Format everything: `mise run fmt`
  - Runs `dprint fmt`
  - Runs `golangci-lint fmt -c .golangci.yml` (gofumpt + goimports)
- Lint: `mise run lint`
  - Verifies config: `golangci-lint config verify -c .golangci.yml`
  - Runs lint: `golangci-lint run ./...`
- Auto-fix (preferred before sending PRs): `mise run fix`
  - Runs `fmt` then `golangci-lint run --fix ./...`

Agents should treat `mise run lint` as a hard gate: if it fails due to your changes, fix it.

## What enforces what

### Go formatting

Formatting is enforced by `golangci-lint fmt` using the formatter configuration in `.golangci.yml`:

- `gofumpt` (with `extra-rules: true`)
- `goimports` (with `local-prefixes: github.com/illumination-k/google-slides-mcp`)

Do not run `gofmt`/`goimports` manually unless you are matching the same configuration.
Prefer `mise run fmt`.

### Go linting

Linting is enforced by `golangci-lint` using `.golangci.yml`.
Notable repo-wide choices:

- `run.tests: true` (lint runs include tests)
- `modules-download-mode: readonly` (avoid implicit `go.mod/go.sum` changes)
- A curated set of linters is enabled (staticcheck, govet, gosec, revive, etc.)

If you add new code:

- Keep APIs minimal and idiomatic Go.
- Prefer explicit error handling.
- Keep functions small and avoid unnecessary allocations.

### Non-Go formatting (docs/config)

`dprint` formats JSON / Markdown / YAML based on `dprint.json`.
If you touch `.md`, `.yml/.yaml`, or `.json`, run `mise run fmt` before finishing.

### GitHub Actions / shell tooling

The mise toolchain also pins:

- `actionlint`
- `shellcheck` (required by actionlint)
- `ghalint`, `pinact`, `zizmor`

If you edit workflow files under `.github/workflows`, run `mise run fmt` and consider running the relevant linters if present.

## Change discipline (agent rules)

## Project layout (layered architecture)

This repo is an application-first Go module. Most code should be organized as:

- `cmd/google-slide-mcp/`: the MCP server entrypoint and transport wiring
- `internal/domain/`: domain types + use-cases (no external I/O)
- `internal/infra/`: infrastructure adapters (Google APIs, HTTP clients, etc.)

Dependency direction:

- `internal/domain` must not import `internal/infra`
- `cmd/...` wires dependencies (dependency injection) and connects transport (MCP) to use-cases
- `internal/infra` can import `internal/domain` to return domain types

### Scope

- Make the smallest change that satisfies the request.
- Avoid drive-by refactors, renames, or stylistic rewrites.
- Do not add new tools or dependencies unless explicitly requested.

### go.mod / go.sum

Because linting uses `modules-download-mode: readonly`, tooling should not mutate modules during lint.

- Do not run `go get` or `go mod tidy` unless the task requires dependency changes.
- If dependency changes are required, ensure `go.mod` and `go.sum` are updated intentionally and that `mise run lint` passes.

### When updating tooling

If you must change formatter/linter behavior or tool versions:

1. Update `.mise.toml`
2. Regenerate lockfile (mise will update `mise.lock`)
3. Ensure `mise run fmt` and `mise run lint` pass

Always commit `.mise.toml` and `mise.lock` together.

## Quick start (for agents)

1. `mise install`
2. Make changes
3. `mise run fix`
4. `mise run lint`

If any step fails, fix before handing off.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/illumination-k)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/illumination-k)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
