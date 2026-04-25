## workspace

> A meta-repository that integrates 5 projects as submodules.

# Aerospike CE Ecosystem — Workspace

A meta-repository that integrates 5 projects as submodules.
Set up the entire development environment in a single step with `git clone --recursive`.

---

## Why This Ecosystem Exists

Aerospike Community Edition (CE) is missing the following tools:

- **No official Kubernetes Operator** — Enterprise AKO exists, but there is no CE Operator
- **No modern Python client** — existing client is C binding (CFFI) based, lacking async/type hints
- **No integrated management UI** — no web management tool for CE clusters
- **No AI development tooling** — no Aerospike-specific AI assistant

This ecosystem fills all four gaps.

---

## Repository Map

| Repo | Path | Tech Stack | Purpose |
|------|------|-----------|---------|
| **aerospike-py** | `aerospike-py/` | Rust/PyO3 + Tokio, Python 3.10-3.14 | High-performance async Python client (2.4x faster than C client) |
| **ACKO** | `aerospike-ce-kubernetes-operator/` | Go 1.25, kubebuilder v4, controller-runtime v0.23 | K8s CE cluster Operator (declarative management via CRD) |
| **Cluster Manager** | `aerospike-cluster-manager/` | FastAPI + Next.js 16, React 19, Tailwind CSS 4 | Web management UI (monitoring, Record browser, Query builder, K8s management) |
| **Plugins** | `aerospike-ce-ecosystem-plugins/` | Claude Code plugin spec | 5 Skills + 1 Agent (AI development support) |
| **Project Hub** | `project-hub/` | Docusaurus v3.9 | Central documentation hub — 40+ ADRs, goals, roadmap, release matrix |

Each repo has its own CLAUDE.md — always refer to it when working in that repo.

---

## Architecture (3 Layers)

```
Layer 1 — Agent + Plugin     : aerospike-ce-ecosystem-plugins (Claude Code)
Layer 2 — Tools              : aerospike-py, ACKO, aerospike-cluster-manager
Layer 3 — Infrastructure     : Aerospike CE on K8s (via ACKO) or bare metal
```

Users manage Layer 3 (infrastructure) through Layer 1 (AI) or Layer 2 (tools).
Each project can be used independently (Loose Coupling). Integration is opt-in.

---

## Dependency & Merge Order

```
aerospike-py → ACKO → cluster-manager → plugins
```

| Repo | Dependencies |
|------|----------|
| **aerospike-py** | standalone — no external dependencies |
| **ACKO** | includes cluster-manager as a submodule (for UI deployment) |
| **cluster-manager** | uses aerospike-py in the backend, integrates with ACKO API for K8s cluster management |
| **plugins** | references API/CRD from all repos as skill content |

### Cross-Repo Impact Scope

- aerospike-py API changes → simultaneous updates to cluster-manager backend + plugins skills
- ACKO CRD changes → simultaneous updates to cluster-manager K8s UI + plugins skills
- Release Compatibility Matrix must be updated on every release (`project-hub/docs/docs/history/releases/`)

---

## CE Constraints (enforced by ACKO Webhook)

| Constraint | Details |
|------|------|
| Max node count | 8 |
| Max Namespace count | 2 |
| XDR | Not available |
| TLS | Not available |
| Security (enterprise auth) | Not available |
| Image | CE only (`aerospike/aerospike-server-enterprise` not allowed) |

CE constraints are validated by the ACKO Webhook at CRD creation time.
Attempting to use Enterprise features produces a clear error message.

---

## Key Documentation

| Document | Path / URL |
|------|-----------|
| **Project Hub (deployed)** | https://aerospike-ce-ecosystem.github.io/project-hub/ |
| **ADR (40+ entries)** | `project-hub/docs/docs/architecture/adr/` |
| **Project Goals** | `project-hub/docs/docs/goals/project-goals.md` |
| **Design Philosophy** | `project-hub/docs/docs/goals/project-design.md` |
| **Q2 2026 Roadmap** | `project-hub/docs/docs/roadmap/current.md` |
| **aerospike-py Docs** | https://aerospike-ce-ecosystem.github.io/aerospike-py/ |
| **ACKO Docs** | https://aerospike-ce-ecosystem.github.io/aerospike-ce-kubernetes-operator/ |

### Key ADRs

| ADR | Decision |
|-----|------|
| PyO3 over CFFI | Memory safety + GIL-free async via Rust/PyO3 |
| Kubebuilder v4 | CRD codegen + envtest support |
| Podman over Docker | Rootless, daemonless container runtime |
| NamedTuple over Dict | IDE autocomplete + type safety |
| Pod Readiness Gates | Zero-downtime rolling update |
| Reconciliation Circuit Breaker | Prevent API server overload |
| IssueOps CI | AI agent-driven automated implementation workflow |

---

## Development Commands

Use the Makefile in this workspace for cross-repo operations:

| Command | Description |
|---------|------|
| `make init` | Initialize all submodules (recursive) |
| `make init-ssh` | Switch submodule URLs to SSH then initialize |
| `make pull-all` | Pull all submodules to latest on main branch |
| `make status` | Check git status of each submodule |
| `make branches` | Check current branch of each submodule |
| `make log-all` | Show last 3 commits for each submodule |
| `make build-py` | Build aerospike-py (Rust + Python) |
| `make test-py` | Run aerospike-py unit tests |
| `make build-acko` | Build ACKO operator |
| `make test-acko` | Run ACKO tests |
| `make test-cm` | Run cluster-manager tests |
| `make lint-all` | Lint all repos |
| `make start-aerospike` | Start local Aerospike CE container |
| `make start-kind` | Create Kind cluster (for ACKO E2E) |

---

## AI Development Skills

### Plugin Installation

```bash
claude plugin install aerospike-ce-ecosystem
```

### Available Skills

| Skill | Trigger | Purpose |
|-------|--------|------|
| `acko-deploy` | "deploy Aerospike on Kubernetes" | CE K8s deployment (includes 8 scenario YAMLs) |
| `acko-operations` | "scale Aerospike cluster" | Day-2 operations (scale, upgrade, dynamic config, troubleshooting) |
| `acko-config-reference` | (background) | CE 8.1 config parameters, CRD mapping, Webhook validation |
| `aerospike-py-api` | "use aerospike-py" | Full Python client API reference |
| `aerospike-py-fastapi` | "FastAPI + Aerospike" | FastAPI production patterns |

### Agent

| Agent | Purpose |
|-------|------|
| `acko-cluster-debugger` | Systematic K8s cluster debugging (6-step diagnostic procedure) |

---

## Conventions

- **Container runtime**: Podman (instead of Docker) — ADR 2026-02-01
- **Language**: English for all documentation; technical terms remain in English
- **Commits**: Conventional Commits (`feat`, `fix`, `refactor`, `docs`, `chore`, `style`, `test`)
- **Branches**: `feature/`, `fix/`, `hotfix/`, `refactor/`
- **Decisions**: Record all architecture decisions as ADRs in project-hub
- **Secrets**: No direct commits of `.env` or credentials — use environment variable references
- **Scope**: Only modify files within the current task scope

---

## Nested Submodule Note

ACKO contains a cluster-manager submodule internally (`aerospike-ce-kubernetes-operator/aerospike-cluster-manager/`).
This is an independent checkout from the `aerospike-cluster-manager/` at the workspace root (different commits are possible).
Resolved automatically with `git clone --recursive`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aerospike-ce-ecosystem) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
