## deezer-tools

> Personal Deezer toolbox in Go. CLI subcommands live under `cmd/deezer-tools/`; reusable packages under `internal/`.

# AGENTS.md

Personal Deezer toolbox in Go. CLI subcommands live under `cmd/deezer-tools/`; reusable packages under `internal/`.

## Documentation layout

- `docs/superpowers/specs/` — design specs for individual features.
- `docs/superpowers/plans/` — implementation plans.
- `docs/superpowers/research/` — protocol and library research notes.
- `docs/solutions/` — documented solutions to past problems (bugs, design patterns, conventions), organized by category with YAML frontmatter (`module`, `tags`, `problem_type`). Relevant when implementing or debugging in documented areas.

Specs, plans, and research live only on `main`. Implementation branches carry only code commits.

## Gateway disclaimer

This repo talks to Deezer's *unofficial* `gw-light.php` gateway, not the documented OAuth API — Deezer no longer accepts new app registrations. The gateway has session-state and JSON-typing quirks that are not obvious from Python OSS references; see `docs/solutions/design-patterns/gw-light-go-adapter-quirks-2026-04-28.md` before adding any new gateway method.

## Auth

`arl` cookie in `~/.config/deezer-tools/config.toml` (mode `0600`). The tool refuses to run if the file is world-readable. Setup steps are in `README.md`.

## Tests

- `go test ./...` runs unit tests; `httptest.Server` is used for gateway request-shape tests.
- `DEEZER_INTEGRATION=1 go test ./internal/gateway/... -run TestIntegration` runs read-only live tests against the real account; skipped by default.

---
> Source: [Frosco/deezer-tools](https://github.com/Frosco/deezer-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
