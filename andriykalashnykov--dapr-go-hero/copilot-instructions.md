## dapr-go-hero

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Educational Go application demonstrating Dapr building blocks (Pub/Sub with routing, Secret Store, State Store, Service Invocation) with four client approaches: custom HTTP, custom gRPC, Go SDK HTTP, and Go SDK gRPC.

## Build, Lint & Test Commands

```bash
make help               # List all available Make targets
make deps               # Install all CLI tools pinned in .mise.toml via mise
make clean              # Remove build artifacts
make static-check       # Composite quality gate: lint-ci + lint + sec + vulncheck + secrets + docker-lint + trivy-fs + trivy-config + mermaid-lint + diagrams-check + deps-prune-check
make lint               # Run golangci-lint (includes gocritic, gosec via .golangci.yml)
make lint-ci            # Lint GitHub Actions workflows (actionlint + shellcheck)
make sec                # Run gosec security scanner (excludes generated proto/ dir)
make vulncheck          # Check dependencies for known vulnerabilities (govulncheck)
make secrets            # Scan for hardcoded secrets (gitleaks)
make trivy-fs           # Filesystem scan for CVEs / secrets / misconfigs (CRITICAL, HIGH)
make trivy-config       # K8s manifest scan for security misconfigurations (KSV-*)
make test               # Run unit tests (go test -race -v ./...)
make integration-test   # Run integration tests (real PostgreSQL + Dapr-state stub + gRPC server)
make build              # Compile both binaries (depends on deps only)
make format             # Auto-format Go source (golangci-lint --fix + gofmt)
make proto-gen          # Regenerate gRPC code from .proto files (mise-pinned protoc)
make update             # Update all deps to latest (go get -u ./... && go mod tidy)
make ci                 # Full CI pipeline: deps → format → static-check → test → integration-test → build
make ci-run             # Run GitHub Actions workflow locally via act
make deps-check         # Show Go version and mise tool status
make deps-prune         # Remove unused and redundant dependencies
make deps-prune-check   # Verify no prunable dependencies (CI gate)
make release            # Tag and push a release (runs full build first)
make docker-build       # Build container images (inventory + products)
make docker-lint        # Lint Dockerfiles with hadolint
make docker-smoke-test  # Boot each image briefly and assert it stays up (Gate 3 of /harden-image-pipeline; SERVICES=name to scope)
make kind-up            # Full stack: create KinD cluster + cloud-provider-kind + Dapr + deploy manifests (alias for kind-deploy)
make kind-down          # Full teardown (alias for kind-destroy)
make kind-create        # Cluster + LoadBalancer controller + Dapr (no app deploy)
make kind-bootstrap     # Install cloud-provider-kind + Dapr into an existing cluster (CI use after helm/kind-action)
make kind-deploy        # kind-create + k8s-deploy (full stack up)
make kind-destroy       # Delete cluster, stop cloud-provider-kind, prune kindccm-* orphan sidecars
make k8s-deploy         # Build images, load into KinD, deploy all manifests
make k8s-undeploy       # Remove all K8s manifests
make k8s-status         # Show pod/service status across all namespaces
make e2e                # Run end-to-end tests (default: sdk-http mode)
make e2e-all            # Run e2e across all 4 client modes (sdk-http, sdk-grpc, custom-http, custom-grpc)
make e2e-setup          # Alias for kind-deploy (CI compatibility)
make e2e-teardown       # k8s-undeploy + kind-destroy (CI compatibility)
```

Run a single test:
```bash
go test ./pkg/features/widgets/... -run TestName -v
```

## Three-Layer Test Pyramid

| Layer | Command | Latency | Scope |
|-------|---------|---------|-------|
| Unit | `make test` | seconds | Pure logic, mocked interfaces, `_test.go` files |
| Integration | `make integration-test` | tens of seconds | Real PostgreSQL via Testcontainers, stub Dapr sidecars, ephemeral-port gRPC servers; gated by `//go:build integration` |
| E2E | `make e2e` / `make e2e-all` | minutes | KinD cluster + cloud-provider-kind + Dapr + full manifest deploy + 11 assertions across 4 client modes |

All three are wired into CI as separate jobs. The unit/integration split keeps the inner-loop fast; the integration layer only runs when changes affect code paths that touch external services.

## Running the Application

Prerequisites: `dapr init` (provides Redis), PostgreSQL container, `widgets` table from `tables.sql`.

```bash
# Terminal 1: Products gRPC service
make run-products

# Terminal 2: Inventory service (pick one client mode)
make run-custom-http    # Custom HTTP client (Fiber), app port 3001
make run-custom-grpc    # Custom gRPC client, app port 4001
make run-sdk-http       # Dapr Go SDK HTTP, app port 3002
make run-sdk-grpc       # Dapr Go SDK gRPC, app port 4002

# Terminal 3: Publish events
make send-widget        # → PostgreSQL via widgets service
make send-gadget        # → Redis State Store via gadgets service
make send-thingamajig   # → gRPC Products service
make send-all           # All three

# Query REST API (Fiber on port 3000)
make get-widget         # GET /v1/widgets/widget
make get-gadget         # GET /v1/gadgets/gadget
make get-thingamajig    # GET /v1/products/thingamajig
```

## Architecture

### Two Binaries

- **`cmd/inventory/main.go`** — Main service: wires up all features, starts Fiber HTTP server (port 3000) for public API, plus Dapr callback servers on separate ports for event handling. Uses `oklog/run` for goroutine lifecycle management. Explicit cleanup (no defers before `os.Exit` calls) to satisfy gocritic `exitAfterDefer`.
- **`cmd/products/main.go`** — Standalone gRPC service for product storage. Default listen address is `127.0.0.1:50151` (loopback for local dev); override to `0.0.0.0:50151` for K8s sidecar reachability via the `PRODUCTS_GRPC_ADDR` env var defined in `pkg/config/config.go`. Invoked via Dapr service discovery.

### Feature Packages (`pkg/features/`)

Each feature follows the repository pattern with `interface.go`, `repository/`, and `service/`:

| Feature | Storage | Event Type | Repository Implementation |
|---------|---------|------------|--------------------------|
| **widgets** | PostgreSQL (pgx) | `widget.v1` | SQL with upsert |
| **gadgets** | Redis State Store | `gadget.v1` | Dapr State API |
| **products** | gRPC Products service | `thingamajig.v1` (default route) | Generated gRPC client |

### Dapr Abstraction Layer (`pkg/dapr/`)

Custom wrappers over Dapr's HTTP/gRPC protocols and the Go SDK:
- `client_http.go` / `client_grpc.go` / `client_sdk.go` — Three client implementations for State and Secret stores
- `server_http.go` / `server_grpc.go` — Event handler registration for callbacks (includes `OnBulkTopicEvent` for bulk pub/sub)
- `subscription.go` — Merges subscriptions from all features into a single Dapr response
- `cloudevent.go` — CloudEvent v1.0 envelope decoding

### Event Flow

CloudEvents published to Dapr PubSub → Dapr routes by `event.type` → appropriate callback handler → feature service → repository → storage backend.

### Key Infrastructure

- **Fiber v3** — Public HTTP API router (port 3000), separate from Dapr callback ports
- **pgx v5** — PostgreSQL driver with connection pooling (`pkg/connect/postgres/`)
- **Logging** — `pkg/log/` provides a minimal `logr.LogSink` backed by zap (replaces `go-logr/zapr`). Used via `log.NewLogger(zapLog)` in `cmd/inventory/main.go`
- **Secrets** — PostgreSQL credentials fetched from Dapr Secret Store (`secrets.json` for local dev)
- **Protobuf** — Service definition in `proto/products/products.proto`, generated code committed
- **gRPC** — Uses `grpc.NewClient` (not deprecated `Dial`/`DialContext`) with `insecure.NewCredentials()`

### Port Map

| Port | Purpose |
|------|---------|
| 3000 | Public Fiber HTTP API |
| 3001 | Custom HTTP Dapr callbacks |
| 3002 | SDK HTTP Dapr callbacks |
| 3500 | Dapr sidecar HTTP |
| 4001 | Custom gRPC Dapr callbacks |
| 4002 | SDK gRPC Dapr callbacks |
| 50151 | Products gRPC service |

All ports are configurable via environment variables defined in `pkg/config/config.go`.

### Kubernetes Deployment

Services deploy into separate namespaces with per-service RBAC:

| Namespace | Contents |
|-----------|----------|
| `dapr-go-hero` | Infrastructure (Redis, PostgreSQL, Zipkin) |
| `dapr-go-hero-inventory` | Inventory deployment + Dapr components (pubsub, statestore, secretstore, subscription, resiliency, configuration) |
| `dapr-go-hero-products` | Products deployment + Dapr configuration (access control) |

Dapr features demonstrated in K8s manifests (`k8s/dapr/`):
- **Pub/Sub** with content-based routing (`subscription.yaml`)
- **State Store** scoped to inventory (`statestore.yaml`)
- **Secret Store** using K8s native secrets (`secretstore.yaml`)
- **Resiliency** — retries, timeouts, circuit breakers (`resiliency.yaml`)
- **Observability** — Zipkin distributed tracing (`configuration.yaml`)
- **Access Control** — app-level RBAC restricting cross-service calls (`configuration.yaml`)

Container images: `Dockerfile.inventory` and `Dockerfile.products` (multi-stage Alpine + BuildKit cache mounts).

Local cluster: KinD + cloud-provider-kind (kind-team-maintained host-side LoadBalancer controller — no in-cluster MetalLB DaemonSet). `make kind-up` creates the full environment. `make kind-destroy` also prunes orphan `kindccm-*` Envoy sidecars (cloud-provider-kind spawns one per LoadBalancer Service; they survive `kind delete cluster` and silently inherit kind-network IPs into the next run if not pruned).

## CI/CD

GitHub Actions CI workflow (`.github/workflows/ci.yml`) runs on every push to `main`, tags `v*`, pull requests, and `workflow_call`:

| Job | Depends on | Steps |
|-----|-----------|-------|
| **changes** | — | Checkout, dorny/paths-filter — gates downstream on whether code/e2e-relevant paths changed |
| **static-check** | changes | Checkout, Set up Go (mise-action), `make static-check` |
| **build** | changes, static-check | Checkout, Set up Go, `make build` |
| **test** | changes, static-check | Checkout, Set up Go, `make test` |
| **integration-test** | changes, static-check | Checkout, Set up Go, `make integration-test` (Testcontainers PostgreSQL) |
| **docker** | changes, static-check, build, test | **Tag-gated** (`refs/tags/v*`). `strategy.matrix.service: [inventory, products]`. Per service: setup-qemu + setup-buildx, docker/metadata-action, single-arch build (`load: true`) → Trivy image scan (CRITICAL/HIGH blocking) → `make docker-smoke-test` → log in to GHCR → multi-arch push (`linux/amd64,linux/arm64`, `provenance: false`, `sbom: false` — Pattern A so GHCR "OS / Arch" tab renders) → cosign keyless OIDC signing by digest |
| **e2e** | changes, build, test | KinD via `helm/kind-action`, `dapr/setup-dapr` CLI install, `make kind-bootstrap`, `make k8s-deploy`, `make e2e-all`, `make k8s-status` on failure |
| **ci-pass** | all above (`if: always()`) | Aggregate gate — single required status check for branch protection; passes when every other job either succeeded or was skipped via path-filter |

The `changes` job uses the canonical path-filter expression so doc-only changes (`**.md`, `docs/**`, `LICENSE`, etc.) skip the heavy jobs but still produce a `ci-pass` status — Repository-Rulesets-safe by default.

A separate cleanup workflow (`.github/workflows/cleanup-runs.yml`) removes old workflow runs weekly.

## Dapr Configuration

- `config.yaml` — Runtime config (tracing via Zipkin, mTLS, metrics)
- `components/` — Component definitions: `pubsub.yaml` (Redis), `statestore.yaml` (Redis, scoped to "inventory"), `secrets.yaml` (local file store), `subscription.yaml` (programmatic subscriptions)
- `secrets.json` — Local dev secrets for PostgreSQL connection

## Protobuf

Regenerate gRPC code after modifying `proto/products/products.proto`:
```bash
make proto-gen
```

The `proto-gen` target pins `protoc`, `protoc-gen-go`, and `protoc-gen-go-grpc` to versions set in `.mise.toml` (installed via `mise install`), so regeneration is reproducible. The target calls `mise exec -- protoc` directly so PATH ordering doesn't matter.

The Dapr cluster runtime version is pinned in the Makefile as `DAPR_RUNTIME_VERSION` (currently `1.17.6`; `make kind-create` and `make kind-bootstrap` pass it to `dapr init -k --runtime-version $(DAPR_RUNTIME_VERSION)`). Keeps local clusters aligned with CI's pinned sidecar version regardless of the developer's Dapr CLI default.

## Static Analysis

`make lint` runs **golangci-lint** (gocritic, gosec, bodyclose, errorlint, goconst, misspell, noctx all configured via `.golangci.yml`).

The full `make static-check` composite gate runs:
- `make lint-ci` — actionlint + shellcheck on `.github/workflows/*.yml`
- `make lint` — golangci-lint
- `make sec` — standalone gosec pass (excludes generated `proto/` dir)
- `make vulncheck` — govulncheck against `go.mod` deps
- `make secrets` — gitleaks
- `make docker-lint` — hadolint on `Dockerfile.inventory` + `Dockerfile.products`
- `make trivy-fs` — filesystem vuln/secret/misconfig scan (CRITICAL, HIGH)
- `make trivy-config` — K8s manifest misconfig scan (KSV-*)
- `make mermaid-lint` — validates every ` ```mermaid ` block via `minlag/mermaid-cli` (same engine GitHub renders with) so broken diagrams don't ship to the README
- `make diagrams-check` — PlantUML drift gate: `make diagrams` must produce output matching what's committed
- `make deps-prune-check` — `go mod tidy` must produce no diff

## Code Quality Conventions

- **Build gate**: `make ci` runs `format → static-check (lint-ci + lint + sec + vulncheck + secrets + docker-lint + trivy-fs + trivy-config + mermaid-lint + diagrams-check + deps-prune-check) → test → integration-test → build`. All must pass with zero issues.
- **gosec**: Generated protobuf files (`proto/`) are excluded. `#nosec` annotations used sparingly with justification.
- **gocritic**: Integrated into golangci-lint via `.golangci.yml` with all tags enabled. No `defer` before `os.Exit` — use explicit cleanup instead.
- **gRPC APIs**: Use `grpc.NewClient` (not `grpc.Dial`/`DialContext`). Use `insecure.NewCredentials()` (not `grpc.WithInsecure()`).
- **Error returns**: All error returns must be checked (`errcheck`). Use `_ =` for intentionally ignored errors in cleanup paths.
- **Parameter style**: Combine consecutive same-type params (`store, key string` not `store string, key string`).
- **Network binding**: gRPC listeners bind to `0.0.0.0` for K8s sidecar reachability, with `#nosec G102` annotations and justification. Listen addresses are configurable via env vars defined in `pkg/config/config.go` (see `cmd/products/main.go` for the loopback-default + env-var-override pattern).
- **kubectl context safety**: every Makefile recipe uses `$(KUBECTL)` (which expands to `kubectl --context=kind-$(CLUSTER_NAME)`) instead of bare `kubectl`. The `tests/e2e.sh` script uses an equivalent `KUBECTL=(kubectl --context=...)` array. This protects against parallel `make` runs from sibling KinD-using projects silently flipping the global current-context mid-recipe.

## Refactor Backlog

Design / refactor TODOs that are intentional, not upstream-blocked. Do not let `/upgrade-analysis` re-flag these as deferred upgrades — they're tech-debt items.

- [ ] Migrate Fiber v3 → `net/http` + `http.ServeMux` (Go 1.22+ pattern routing). Rationale: Fiber is built on fasthttp which is incompatible with `net/http.Handler`, forcing custom adapters for OTel instrumentation, `httptest`, and standard tracing propagators. The inventory binary already runs three other servers on stdlib (Dapr SDK HTTP + SDK gRPC + raw gRPC), so Fiber is a lone outlier. Scope: `cmd/inventory/main.go` (drop `fiber.Config`, replace `app.Listen` / `dapr.RegisterServices` / `dapr.RegisterEventHandlers` / `dapr.SubscribeHTTPHandler` plumbing with `http.ServeMux`), all `pkg/features/*/service/service.go` handlers (`fiber.Ctx` → `http.ResponseWriter, *http.Request`), `pkg/dapr/server_http.go` and `pkg/dapr/client_http.go` (the Fiber HTTP client in `client_http.go` can move to `net/http` too — `client_http_integration_test.go` already stubs via `net/http/httptest`). Consider `go-chi/chi` as a drop-in if stdlib mux patterns feel thin. Removes one dependency and the fasthttp compatibility tax.

## Upgrade Backlog

Items surfaced by `/upgrade-analysis` that are upstream-blocked or awaiting a named trigger.

- [ ] Track `pgx/v5` CVEs GO-2026-4772 / GO-2026-4771 — **Fixed in: N/A upstream** (verified 2026-04-12). Code not reachable per govulncheck. Upgrade when patched version is released.
- [ ] Monitor `dapr/setup-dapr@v2` — runs on Node.js 20; GitHub forces Node 24 starting 2026-06-02. Upstream tag `v2` is still July 2024; awaiting Node.js 24 compatible release before Renovate can bump SHA.
- [ ] OTel migration: replace Zipkin-direct tracing once Dapr deprecates the Zipkin exporter (no EOL announced yet).

*Last reviewed: 2026-05-05*

## Skills

Use the following skills when working on related files:

| File(s) | Skill |
|---------|-------|
| `Makefile` | `/makefile` |
| `renovate.json` | `/renovate` |
| `README.md` | `/readme` |
| `CLAUDE.md` | `/claude` |
| `.github/workflows/*.{yml,yaml}` | `/ci-workflow` |
| `docs/diagrams/*.puml`, Mermaid blocks in markdown | `/architecture-diagrams` |
| Dockerfiles + image publishing | `/harden-image-pipeline` |
| Test layers (`*_test.go`, `tests/e2e.sh`) | `/test-coverage-analysis` |

When spawning subagents, always pass conventions from the respective skill into the agent's prompt.

---
> Source: [AndriyKalashnykov/dapr-go-hero](https://github.com/AndriyKalashnykov/dapr-go-hero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
