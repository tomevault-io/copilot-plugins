## ogou

> Guidance for AI coding agents and human contributors. The project constitution (`.specify/memory/constitution.md`) is the highest-authority development document.

# Ogou — Agent & Contributor Guide

Guidance for AI coding agents and human contributors. The project constitution (`.specify/memory/constitution.md`) is the highest-authority development document.

## What this is

HTTP load and smoke testing CLI. Generic, open-source, protocol-agnostic engine with an HTTP driver.

- **Module:** `github.com/madstone-tech/ogou` · **Binary:** `htload`
- **Go:** 1.25+ · Single static binary, zero runtime dependencies

## Build & test (Taskfile)

```bash
task build        # go build ./cmd/htload → ./htload
task test         # go test ./...
task test-pkg     # go test -race -cover ./pkg/...
task security     # gosec (blocking) + govulncheck (advisory)
task lint         # golangci-lint
task fmt          # gofumpt
task ci           # fmt + lint + test + security + build
task release-snapshot  # goreleaser snapshot
task docker:build # docker build -t ogou:latest .
```

## Architecture

Two-layer CLI: **quick mode** (`htload <url>`) and **scenario mode** (`htload -f file.yaml`).

```
pkg/engine/          Core: Scenario, Runner, Driver interface, Reporter interface
pkg/http/            HTTP/1.1+2 driver (implements Driver)
pkg/reporter/        Console, JSON, Discard reporters
pkg/scenario/        YAML parser + validator
pkg/auth/            Auth provisioners (bearer, apikey, basic, oauth-cc)
internal/template/   text/template funcmap (UUID, RandomString, Timestamp, Env)
internal/metrics/    Latency aggregation, percentile calculation
internal/capture/    Response extractors (jsonpath, header, regex)
internal/assertion/  Assertion engine (status, jsonpath, response_time)
internal/driver/     Driver registry (extensible: WebSocket, gRPC future)
cmd/htload/          CLI entry (Cobra, two-file pattern)
tests/               Integration tests (httptest server)
examples/            Sample scenario YAMLs
```

**Protocol-agnostic core** — `pkg/engine/` must NOT import `net/http`. HTTP is a driver.

## Conventions

- **Format:** `gofumpt`; **lint:** `golangci-lint` — both zero-warning, enforced in CI.
- **Two-file command pattern** (Principle II): `cmd_cobra.go` (flags) + `cmd.go` (logic).
- **No silent error swallowing** (Principle IV): every error checked or commented.
- **Stdlib first** (Principle III): new deps need justification in plan Complexity Tracking.
- **Coverage:** `pkg/` ≥ 80%, `cmd/` ≥ 60%.

## Development Workflow (Speckit)

Feature specs live in `specs/<###-name>/`:

1. `/speckit-specify` — generate `spec.md`
2. `/speckit-plan` — generate `plan.md`
3. `/speckit-tasks` — generate `tasks.md`
4. Implement
5. `/speckit-analyze` — verify spec/plan/tasks consistency

Prompt seeds (reference material) live in `prompts/<###-name>/seed.md`.

Constitution check gates every phase. See `.specify/memory/constitution.md`.

---
> Source: [madstone-tech/ogou](https://github.com/madstone-tech/ogou) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
