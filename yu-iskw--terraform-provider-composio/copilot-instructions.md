## terraform-provider-composio

> Shared guidance for agents working in this repository.

# AGENTS.md

Shared guidance for agents working in this repository.

## Overview

This is a Terraform provider for Composio written in Go with the HashiCorp Terraform Plugin Framework. It manages durable Composio control-plane objects. It does not model user OAuth, sessions, or tool execution.

## mise (optional)

If you use [mise](https://mise.jdx.dev/), run `mise trust` in the repo root on first use if mise refuses to load the config, then `mise install`. [`mise.toml`](mise.toml) installs **only the Trunk CLI** (pinned to match `.trunk/trunk.yaml` `cli.version`). **Go is not installed via mise**; use your normal Go install and the `go` / `toolchain` lines in `go.mod`. When you upgrade Trunk, bump `cli.version` in `.trunk/trunk.yaml` and the `trunk` version in `mise.toml` together.

## Key Commands

- Unit tests: `make test`
- Acceptance tests: `make testacc` (requires `COMPOSIO_API_KEY`; see CONTRIBUTING.md)
- Build: `go build -v ./`
- Generate docs: `go generate ./...`
- Format: `make format`
- Lint: `make lint`

## Project Structure

- `main.go`: provider server entry point
- `internal/provider`: provider schema, configuration, resources, data sources, embedded docs, and tests
- `internal/composio/api`: Composio REST client (v3.1, typed errors, retries, org/project headers)
- `internal/composio/models`: domain models without Terraform types
- `examples`: Terraform examples used by docs generation
- `docs`: provider documentation
- `tools`: Go tool dependencies
- `mise.toml`: optional Trunk CLI version for mise users (no Go via mise)

## Development Notes

- Prefer real, deterministic tests over mocks. HTTP tests use `httptest.Server`.
- Do not wrap runtime Composio APIs (tool execute, sessions, connect links) as resources.
- Git hooks for this repo are **Trunk-only** (`make setup-dev` runs `trunk git-hooks sync`); there is no separate pre-commit install step.

---
> Source: [yu-iskw/terraform-provider-composio](https://github.com/yu-iskw/terraform-provider-composio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
