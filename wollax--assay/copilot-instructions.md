## assay

> You are working within the Assay monorepo, an agentic development kit built in Rust. The repo contains two projects: **assay** (spec-driven dev kit) and **smelt** (infrastructure layer for container-based job execution).

# Assay — Agent Instructions

You are working within the Assay monorepo, an agentic development kit built in Rust. The repo contains two projects: **assay** (spec-driven dev kit) and **smelt** (infrastructure layer for container-based job execution).

## Build Commands

- `just build` — build all crates (both projects)
- `just test` — run all tests (both projects)
- `just lint` — run clippy (both projects)
- `just fmt` — format code
- `just ready` — run all checks (fmt, lint, test, deny, check-plugin-version)
- `just build-assay` — build assay crates only
- `just build-smelt` — build smelt crates only
- `just test-assay` — test assay crates only
- `just test-smelt` — test smelt crates only (Docker integration tests skip gracefully when Docker is unavailable)
- `just test-smelt-unit` — test smelt crates only, explicitly excluding `docker_lifecycle` integration tests
- `just lint-assay` — lint assay crates only
- `just lint-smelt` — lint smelt crates only

## Workspace Layout

Root `Cargo.toml` declares `members = ["crates/*", "smelt/crates/*"]`.

### Assay crates (`crates/`)

- `crates/assay-types` — shared serializable types (serde, schemars)
- `crates/assay-core` — domain logic (specs, gates, reviews, workflows)
- `crates/assay-backends` — state backend implementations (Linear, GitHub, Smelt, SSH)
- `crates/assay-harness` — single-agent harness for running specs
- `crates/assay-mcp` — MCP server with signal endpoint
- `crates/assay-cli` — CLI binary (clap)
- `crates/assay-tui` — TUI binary (ratatui)

### Smelt crates (`smelt/crates/`)

- `smelt/crates/smelt-core` — infrastructure layer: Docker/Compose/K8s job execution, tracker, forge delivery
- `smelt/crates/smelt-cli` — daemon binary with TUI, HTTP API, SSH worker pools, queue persistence

### Other

- `plugins/` — plugin packages for agentic AI systems (claude-code, opencode, smelt-agent)

## Cross-Project Dependencies

- `smelt-core` depends on `assay-types` via path dep (`path = "../../../crates/assay-types"`) for `StateBackendConfig` and related types
- Changes to `assay-types` may require corresponding updates in smelt-core

## Conventions

- Lean towards functional and declarative patterns
- Use workspace dependencies defined in the root `Cargo.toml`
- Types shared between crates belong in `assay-types`
- Business logic belongs in `assay-core`
- Binary crates are thin wrappers that delegate to `assay-core`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wollax)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wollax)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
