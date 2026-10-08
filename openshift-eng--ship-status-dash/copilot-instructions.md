## general

> Ship Status Dashboard repository overview and general coding guidelines


### APM context generation

Sources under **`.apm/`** (instructions, prompts, `apm.yml`, etc.) drive generated agent context. After editing those files, run **`make apm`** to regenerate **AGENTS.md**, **CLAUDE.md**, **GEMINI.md**, and the integrated copies under **`.claude/`**, **`.cursor/`**, **`.gemini/`**, and **`.opencode/`**. CI enforces freshness with **`make verify-apm`**.

**Slash / agent commands:** content under **`.apm/prompts/*.prompt.md`** is the single source of truth. **`apm install`** (part of **`make apm`**) copies each prompt into editor command targets (e.g. **`.claude/commands/`**, **`.opencode/commands/`**, **`.gemini/commands/`**). Do not add those generated paths by hand or installs will skip them as unmanaged duplicates.

### Overview

**Ship Status Dashboard** is a status monitoring and availability tracking system for OpenShift CI components. It provides:

* Real-time component status and outage tracking
* Automated monitoring via HTTP probes, Prometheus queries, and systemd checks
* Slack integration for outage notifications
* Audit logging for all outage modifications

The system consists of:

* A **Go-based API backend** with dual-ingress auth (public + OAuth-protected routes)
* A **React/Vite/Material-UI frontend** (located in `frontend/`)
* **Component-monitor** services that probe infrastructure components and report to the dashboard
* **MCP servers** that expose dashboard API tools to AI agents (a public read-only instance and an authenticated write instance behind oauth-proxy)
* **PostgreSQL** for data storage

### Deployment topology

Deployment manifests live in the [openshift/release](https://github.com/openshift/release) repository under `clusters/`. Configuration files (component definitions, monitor configs) live in `core-services/ship-status/` in that same repo.

**app.ci cluster** (`clusters/app.ci/ship-status-dash/`):

* **Dashboard pod** -- runs in the `ship-status` namespace with six containers:
  - `dashboard` (port 8080) -- Go API backend, reads config from openshift/release via git-sync
  - `oauth-proxy` (port 8443) -- authenticates requests to the dashboard via OpenShift OAuth / Kubernetes TokenReview, signs with HMAC
  - `oauth-proxy-mcp` (port 9443) -- authenticates requests to the write MCP via the same mechanism
  - `ship-status-mcp` (port 8090) -- public read-only MCP server for AI agents
  - `ship-status-mcp-auth` (port 8091) -- authenticated write MCP server, behind oauth-proxy-mcp
  - `git-sync` -- continuously syncs the openshift/release repo for config
* **Component-monitor pod** (`app-ci-component-monitor`) -- probes Prow services (deck, crier, sinker, tide, gangway), Boskos pools, and build farm clusters (build01-11). Reports results to the dashboard's protected API using its own service account token.
* **Ingress routes:**
  - `ship-status.ci.openshift.org` -- public read-only API (port 8080)
  - `protected.ship-status.ci.openshift.org` -- OAuth-protected dashboard API (port 8443)
  - `mcp.ship-status.ci.openshift.org` -- public read-only MCP server (port 8090)
  - `protected-mcp.ship-status.ci.openshift.org` -- OAuth-protected write MCP (port 9443)

**DPCR cluster:**

* **Component-monitor instance** (`dpcr-component-monitor`) -- monitors Sippy components (Sippy, Sippy-Auth, Sippy Chat). Runs on the DPCR cluster but reports back to the same dashboard on app.ci. Has its own service account and non-expiring token for cross-cluster auth.

**Build clusters** (`clusters/build-clusters/build-shared/ship-status/`):

* **Service account and RBAC** for component-monitor to access Prometheus and routes on build farm clusters. The app.ci component-monitor uses kubeconfigs for these clusters to run remote probes.

Favor clarity and maintainability over cleverness. Comments should be minimal, helpful, and explain the "why" not the "what".

---
> Source: [openshift-eng/ship-status-dash](https://github.com/openshift-eng/ship-status-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-08 -->
