## hub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Analytics Hub — a corporate analytics platform built on JupyterHub that provides isolated user workspaces with Hugr-connected Jupyter kernels (GraphQL, DuckDB, Python), an AI agent assistant, interactive data visualizations (Dives), and task scheduling. Part of the hugr-lab ecosystem.

## Architecture

Three-layer architecture: **Control Plane** (JupyterHub + Hub Service) → **User Plane** (workspace + agent containers per user) → **Data Plane** (Hugr server, external).

**Two auth paths:**
- Workspace (interactive): OIDC tokens, refreshed by polling JupyterHub API. Refresh token never enters containers.
- Agent/scheduler (programmatic): Hub Service uses Hugr management secret (`x-hugr-secret-key`) with user identity headers. No token expiry.

**Hub Service** is a Go service built on `airport-go` that registers with Hugr as an Arrow Flight data source. Hub metadata (agent memory, query registry, dives, scheduler) is accessible via standard Hugr GraphQL API.

**Agent** runs in a separate container with sandboxed tools (Python, bash). Connects to Hub Service as MCP client — has no credentials. Hub Service adds auth to all Hugr requests.

**OIDC auto-discovery**: Hub needs only `HUGR_URL` (base, without `/ipc`) + `OIDC_CLIENT_SECRET` + `HUB_BASE_URL`. Discovers OIDC endpoints from `GET {HUGR_URL}/auth/config` → `{issuer}/.well-known/openid-configuration`.

## Key Conventions

- `HUGR_URL` is the base URL (e.g., `http://hugr:15000`). Kernel IPC endpoint is `{HUGR_URL}/ipc` — appended automatically by connection service.
- Managed connections (`managed: true` in connections.json) are created by the Hub, immutable by users.
- Design documents live in `design/` (gitignored, local only).
- Hub Service Go code will live in `cmd/` and `pkg/` directories.
- Docker images: `Dockerfile.hub` (JupyterHub), `Dockerfile.singleuser` (workspace), `Dockerfile.hub-service` (Go service), `Dockerfile.hub-agent` (agent).
- `HUGR_SPAWNER` env var selects spawner: `docker` (default, DockerSpawner) or `kubernetes` (KubeSpawner).
- Helm chart for K8s deployment lives in `k8s/hugr-hub/`. Uses z2jh as subchart.
- For minikube: use `host.minikube.internal` to reach host services (Hugr, Keycloak).

## Related Repositories (~/projects/hugr-lab/)

- `hugr` — Core DataMesh server (Go). GraphQL API, DuckDB engine, auth, OIDC validation.
- `query-engine` — GraphQL query engine library (Go). Auth middleware, RBAC, permissions.
- `hugr-kernel` — Jupyter kernel for Hugr GraphQL (Go + Python). Connection service with OIDC token management. **Key integration point for Stage 1** — needs hub mode + managed connections.
- `duckdb-kernel` — DuckDB Jupyter kernel (Go).
- `hugr-client` — Python client for Hugr IPC protocol.
- `airport-go` — Arrow Flight server library (Go). Used to build Hub Service.
- `mcp` — MCP server exposing Hugr data sources to AI agents (Go).

## Implementation Stages

1. **JupyterHub + OIDC + Kernels** — DockerSpawner, OIDC auto-discovery from Hugr, managed connections, token refresh via JupyterHub API polling (spec in `design/001-analytics-hub/stage-1-spec.md`)
2. **Hub Service** — Go/airport-go, Flight server for hub.* data, Agent Manager (Docker/K8s), per-user MCP endpoints
3. **Agent Runtime** — Sandboxed container, MCP client to Hub Service, skills (SKILL.md), memory (local + pgvector), tool policies
4. **Dives** — React data apps, hugr-dive-sdk, viewer-identity query proxy, JupyterLab WebView preview
5. **Scheduler** — User-level (in workspace), system-level (Hub Service, management auth)

## Active Technologies
- Python 3.12+ (JupyterHub config, connection service), Go 1.26.1 (kernel changes in hugr-kernel repo) + jupyterhub 4.x, oauthenticator, dockerspawner, httpx (Hub); hugr_connection_service (workspace) (001-jupyterhub-oidc-workspace)
- Docker named volumes for user data persistence; connections.json for token state (001-jupyterhub-oidc-workspace)
- Python 3.12+ (JupyterHub config, server extensions), Go 1.26.1 (hub-service future) + `jupyterhub 5.x`, `dockerspawner`, `oauthenticator`, `jupyterhub-idle-culler`, `hugr-client>=0.2.0` (002-resource-limits)
- profiles.json (file-based config), Docker named volumes, NFS, S3/Azure/GCS FUSE (002-resource-limits)
- Python 3.12+ (JupyterHub config, hub_profiles module), Helm/YAML (chart templates) + z2jh Helm chart 4.x (subchart), kubespawner, jupyterhub 5.x, oauthenticator, httpx (003-k8s-deployment)
- PVC (user home), ConfigMap (profiles.json), K8s Secrets (credentials), FUSE init containers (S3/Azure on minikube) (003-k8s-deployment)
- Go 1.26.1 (Hub Service + Agent binary), TypeScript (JupyterLab extensions) + `query-engine/client/app` (hugr-app SDK), `airport-go`, `mark3labs/mcp-go`, `query-engine/client` (Hugr GraphQL client) (004-hub-service-agent)
- PostgreSQL 16 + pgvector (managed by Hugr via app framework) (004-hub-service-agent)
- Go 1.26.1 (Hub Service, Agent), TypeScript (JupyterLab extensions), Python 3.12+ (server extensions) + `airport-go`, `query-engine/client`, `mark3labs/mcp-go v0.47.0`, `nhooyr.io/websocket`, `@jupyterlab/apputils`, `@lumino/widgets` (005-chat-agent-io)
- PostgreSQL + pgvector (via Hugr App framework — conversations, messages tables) (005-chat-agent-io)
- Go 1.26.1 (Hub Service, Agent), TypeScript (JupyterLab extensions), Python 3.12+ (JupyterHub config) + `query-engine/client` v0.3.22 (upgrade from v0.3.20), `airport-go` v0.2.1, `mcp-go` v0.47.0, `nhooyr.io/websocket` v1.8.17 (006-streaming-impersonation)
- PostgreSQL 16 + pgvector (via Hugr App framework — all access through Hugr GraphQL, never direct DB) (006-streaming-impersonation)
- Go 1.26.1 (hub-service, hub-agent), Python 3.12+ (JupyterHub config), TypeScript (JupyterLab extensions) + `airport-go` (Arrow Flight), `query-engine/client` (Hugr GraphQL), `nhooyr.io/websocket`, `@jupyterlab/apputils`, `@lumino/widgets` (007-spec-f-foundation)
- PostgreSQL + pgvector via Hugr (all DB access through Hugr GraphQL, never direct — Constitution I), persistent disk volumes for agent state (`$HUB_STORAGE_PATH`) (007-spec-f-foundation)
- Go 1.26.1 (hub-service, hub-agent, kernel-mcp, result-store-mcp), Python 3.12+ (agent-bridge, jupyterhub config), TypeScript (hub-chat extension) + `airport-go`, `query-engine/client` v0.3.25, `mcp-go` v0.47.0, `nhooyr.io/websocket`, `apache/arrow-go/v18`, DuckDB (embedded, for result.query) (008-spec-g-unified-runtime)
- PostgreSQL + pgvector via Hugr (all DB access through Hugr GraphQL — HC1), persistent disk (`$HUB_STORAGE_PATH`) for results/skills/checkpoints (008-spec-g-unified-runtime)

## Recent Changes
- 001-jupyterhub-oidc-workspace: Added Python 3.12+ (JupyterHub config, connection service), Go 1.26.1 (kernel changes in hugr-kernel repo) + jupyterhub 4.x, oauthenticator, dockerspawner, httpx (Hub); hugr_connection_service (workspace)

---
> Source: [hugr-lab/hub](https://github.com/hugr-lab/hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
