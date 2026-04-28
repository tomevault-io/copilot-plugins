## humr

> Humr — a Kubernetes platform for running AI agent harnesses (Claude Code, Codex, Gemini CLI) in isolated environments with credential injection, network isolation, and scheduled execution.

## Project Overview

Humr — a Kubernetes platform for running AI agent harnesses (Claude Code, Codex, Gemini CLI) in isolated environments with credential injection, network isolation, and scheduled execution.

Always check [docs/architecture.md](docs/architecture.md) for full architecture details.

### Monorepo Structure

pnpm workspaces + standalone Go module:
- `packages/agent-runtime/` — ACP WebSocket server + trigger watcher inside agent pods
- `packages/agent-runtime-api/` — tRPC router definitions for agent-runtime
- `packages/humr-base/` — shared base image/utilities
- `packages/example-agent/` — example agent configuration
- `packages/controller/` — Go K8s reconciler + scheduler
- `packages/ui/` — React chat interface (Vite)
- `deploy/helm/humr/` — Helm chart for all components + OneCLI + PostgreSQL

Together, `agent-runtime` + `agent-runtime-api` form the agent runtime — the ACP WebSocket server that runs inside each agent pod.

## Workflow

mise is the task runner. All tasks are defined in `tasks.toml` files. **Always use `mise run` for building, checking, testing, and cluster operations — never invoke `go`, `pnpm`, `helm`, `kubectl`, etc. directly.** mise manages tool versions and environment; running tools directly will break.

```sh
mise run check              # lint + type-check all packages (also runs as pre-commit hook)
mise run test               # run all tests
mise run helm:check:lint    # helm lint
mise run helm:check:render  # helm template | kubeconform
mise run ui:run             # start UI dev server
```

### Cluster lifecycle (k3s via lima)

```sh
mise run cluster:install      # create k3s VM, build images, install cert-manager + Humr chart
mise run cluster:build-agent  # rebuild agent image only, restart agent pods
mise run cluster:status       # show pods and cluster state
mise run cluster:logs         # show OneCLI pod logs
mise run cluster:stop         # stop k3s VM (preserves data)
mise run cluster:uninstall    # helm uninstall + cleanup PVCs
mise run cluster:delete       # destroy k3s VM entirely
```

Services are available at `*.localhost:4444` automatically (Traefik on port 4444, auto-forwarded by lima). `*.localtest.me:4444` also works as an alias.

### Cluster debugging (pre-approved in .claude/settings.json)

Use `mise run cluster:kubectl -- <args>` and `mise run cluster:shell -- <cmd>` instead of raw `kubectl` or `export KUBECONFIG=...`. These are auto-approved.

Activate cluster environment for interactive use: `export KUBECONFIG="$(mise run cluster:kubeconfig)"`.

## Architecture

Three-tier K8s platform:
1. **Controller** (Go) — watches ConfigMaps, reconciles StatefulSets/Services/NetworkPolicies, runs cron scheduler
2. **API Server** (TypeScript) — REST CRUD for instances/templates/schedules, WebSocket ACP relay to agent pods
3. **Agent Runtime** (TypeScript, `agent-runtime` + `agent-runtime-api`) — ACP WebSocket server inside agent pods

Infrastructure:
- **OneCLI** — credential injection proxy (MITM), single container with Rust gateway + Node.js web dashboard
- **cert-manager** — generates self-signed ECDSA CA (PKCS8) for OneCLI MITM TLS
- **PostgreSQL** — OneCLI's internal dependency

K8s resource model: ConfigMaps with `humr.ai/type` labels (agent-template, agent-instance, agent-schedule). Each ConfigMap uses `spec.yaml` (API Server writes) and `status.yaml` (Controller writes) keys to avoid write contention. Not CRDs — deployable without cluster-admin.

## Key Design Decisions

Architecture Decision Records live in [`docs/adrs/`](docs/adrs/) — see [`docs/adrs/index.md`](docs/adrs/index.md) for the full list. Use the `/adr` skill to create and manage ADRs. Always check existing ADRs before proposing architectural changes.

## Commit Conventions

- **Conventional Commits**: `type(scope): short summary` — types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `revert`, `style`, `perf`, `ci`, `build`.
- **Scope**: Optional but encouraged (e.g., `feat(ui):`, `fix(hook):`, `docs(design):`).
- **Body**: Optional concise bullet points for non-trivial changes.
- **Trailer**: Configured via `.claude/settings.json` `attribution` — do not add manually.
- **DCO**: Always use `git commit -s` to add `Signed-off-by` trailer.
- **Branch naming**: `type/short-description` (e.g., `feat/session-history`, `fix/stale-timer`). Same type prefixes as commits.

## Separation of Concerns & DRY Principle

This system is a modular component system following the DRY (Don't Repeat Yourself) principle. Each piece has a single responsibility. You should be able to swap out any component without rewriting others.

## Worktrees

Use `.worktrees/` for git worktrees. Branch naming follows commit conventions (e.g., `feat/session-history`).

### Setup

After creating a worktree, run project setup:

- **Node.js**: `pnpm install`

### Verification

Run tests to confirm a clean baseline before starting work. If tests fail, report failures and ask before proceeding.

### Report

After setup, report: worktree path, test results, and readiness.

## TSEng

This project follows the [TypeScript Engineering](tseng/index.md) architecture.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kagenti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
