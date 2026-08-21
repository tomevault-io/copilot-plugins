## troubleshooting-scenarios

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

This repository contains evaluation suites for AI-assisted troubleshooting on OpenShift. Each eval suite owns a top-level directory with scenarios that deploy faults on a live cluster, send queries to OpenShift Lightspeed (OLS), and score responses with a judge LLM using the [lightspeed-evaluation](https://github.com/lightspeed-core/lightspeed-evaluation) framework.

The `generic/` directory contains standalone fault-injection demos that do not use the eval framework.

## Repository Structure

```
scripts/          Shared shell scripts and eval.mk (MCP, OLS, venv, port-forward)
_template/        Copyable skeleton for new eval suites
kiali-ossm/       Kiali/OSSM service-mesh evaluation scenarios
netobserv/        NetObserv network observability evaluation scenarios
generic/          Standalone troubleshooting demos (deploy/break/fix lifecycle)
```

## Working with Eval Suites

All commands run from within a suite directory (e.g., `cd kiali-ossm/`).

### Prerequisites

- `oc login` to an OpenShift 4.x cluster
- `OPENAI_API_KEY` exported

### Lifecycle Commands

```bash
make setup     # Install venv + OLS operator + MCP server + suite dependencies
make evals     # Run all scenarios (auto port-forward to OLS)
make cleanup   # Remove suite dependencies + MCP server (OLS stays)
```

Run a single scenario:

```bash
make check_mesh_status-eval
```

### How It Works

Each suite Makefile declares a `SCENARIOS` variable and includes `../scripts/eval.mk`, which auto-generates `<scenario>-eval` targets. The shared `run-evals.sh` script handles system.yaml URL replacement, port-forward lifecycle, and `lightspeed-eval` invocation.

### Key Files per Team

- `Makefile` — declares SCENARIOS, MCP config, setup/cleanup targets
- `system.yaml` — evaluation framework config (judge model, metrics, output)
- `evals.yaml` — conversation definitions (queries + expected responses, with tags matching SCENARIOS)
- `build/` — suite-specific setup scripts and cluster resources
- `<scenario>/setup.sh` — runs before the conversation (deploy fixtures)
- `<scenario>/cleanup.sh` — runs after (remove fixtures)
- `<scenario>/fixtures/` — Kubernetes manifests

### Shared Scripts (scripts/)

| Script | Purpose |
|--------|---------|
| `eval.mk` | Makefile include: target generation, _setup-shared, _cleanup-shared |
| `setup-venv.sh` | Create venv with lightspeed-eval (idempotent) |
| `setup-ols.sh` | Install OLS operator + OLSConfig (idempotent) |
| `setup-mcp.sh` | Deploy MCP server with configurable toolsets |
| `connect-ols-mcp.sh` | Register MCP in OLSConfig + restart + wait |
| `run-evals.sh` | Port-forward + lightspeed-eval + cleanup |

### Root Makefile

The root Makefile has maintenance targets only:

```bash
make cleanup        # Remove OLS operator + local venv
```

## Architecture: generic/01-payments-api-failure (Database Connection Exhaustion)

Services share a PostgreSQL database with `max_connections=20`. The fault: `reporting-service` v1.0.2 accumulates database connections without closing them, exhausting the shared pool and causing payments-api to return 503s.

Two deployment modes control difficulty:

- **`make deploy-easy`**: single `payments` namespace, per-service DB users (`payments`, `reporting`), only 1 critical alert (payment error rate) and 1 warning (DB connections), no red herring.
- **`make deploy`** (hard): two namespaces (`payments`, `shared-services`), shared `dbuser` DB account, graduated alerts (warning + critical for both payment and DB), `reconciliation-service` red herring (always in CrashLoopBackOff). Accepts `SINGLE_NAMESPACE=1` to collapse into one namespace while keeping other hard-mode traits.

The deploy script always works on a temp copy of manifests, applying sed transformations per mode. Operational scripts (break, fix, cleanup, etc.) auto-detect the deployment mode by checking whether the `shared-services` namespace exists.

### Key Paths

- `generic/01-payments-api-failure/README.md` -- scenario overview and components
- `generic/01-payments-api-failure/manifests/payments/` -- Kubernetes manifests for the payments namespace
- `generic/01-payments-api-failure/manifests/shared-services/` -- Kubernetes manifests for the shared-services namespace
- `generic/01-payments-api-failure/scripts/` -- shell scripts that implement each Make target
- `generic/01-payments-api-failure/reporting-service/v1.0.1/` -- healthy version
- `generic/01-payments-api-failure/reporting-service/v1.0.2/` -- buggy version (connection leak + division by zero)

## Architecture: generic/02-alert-storm (Cascading Alert Storm)

Single `payments` namespace with five microservices:

- **`payments-api`**: Central service that processes payments. Loads config from a mounted ConfigMap.
- **`checkout-service`**, **`order-processor`**, **`refund-service`**, **`notification-service`**: Downstream services that depend on `payments-api` via HTTP.

The fault: A broken ConfigMap is applied to `payments-api`, causing it to fail. All four downstream services degrade in cascade, triggering a storm of alerts that obscures the simple root cause.

Monitoring is wired via Prometheus ServiceMonitors and PrometheusRules with alerts on error rates, latency, queue depths, and resource usage across all five services.

### Key Paths

- `generic/02-alert-storm/README.md` -- scenario overview and components
- `generic/02-alert-storm/manifests/` -- Kubernetes manifests (namespace, deployments, ServiceMonitors, PrometheusRules)
- `generic/02-alert-storm/manifests/configmaps/` -- healthy and broken ConfigMap variants
- `generic/02-alert-storm/scripts/` -- shell scripts that implement each Make target
- `generic/02-alert-storm/images/` -- Dockerfiles and Python source for all five services

## Architecture: generic/03-image-pull-failure (Image Pull Failure with PDB Alert)

Single `inventory` namespace with one application:

- **`inventory-app`**: A simple service using Red Hat UBI (`registry.redhat.io/ubi9/ubi`) with 3 replicas. A PodDisruptionBudget requires at least 2 healthy pods.

The fault: A typo is introduced in the container image reference (`ubi9/ubi9` instead of `ubi9/ubi`), causing all pods to enter `ImagePullBackOff`. With zero healthy pods, the PDB is violated and the platform-level `PodDisruptionBudgetLimit` alert fires.

No custom images or PrometheusRules are needed — this scenario relies on a standard Red Hat image and the built-in OpenShift PDB alert.

### Key Paths

- `generic/03-image-pull-failure/README.md` -- scenario overview and components
- `generic/03-image-pull-failure/manifests/` -- Kubernetes manifests (namespace, deployment, service, ServiceMonitor, PDB)
- `generic/03-image-pull-failure/scripts/` -- shell scripts that implement each Make target

## Tech Stack

- **Eval framework**: [lightspeed-evaluation](https://github.com/lightspeed-core/lightspeed-evaluation), Python 3.11–3.13
- **System under test**: [OpenShift Lightspeed](https://github.com/openshift/lightspeed-service) with MCP server
- **Applications** (generic scenarios): Python 3.12, FastAPI, raw psycopg2
- **Infrastructure**: OpenShift 4.x, Prometheus user workload monitoring
- **Deployment**: Raw Kubernetes YAML manifests via `oc apply`, no Helm/Kustomize

---
> Source: [rhobs/troubleshooting-scenarios](https://github.com/rhobs/troubleshooting-scenarios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
