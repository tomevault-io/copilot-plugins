## qubership-redis

> Guidance for agents when working in this repository.

# AGENTS.md

Guidance for agents when working in this repository.

## Repository Overview

Kubernetes operator for Redis on Kubernetes/OpenShift. Multi-module Go workspace, three components:

- **redis-operator** — main Kubernetes operator (Operator SDK, controller-runtime)
- **redis-monitoring-agent** — Telegraf-based sidecar for metrics
- **redis-tests** — RobotFramework integration test suite

Run single test:
```bash
cd redis-operator && go test ./path/to/package/... -run TestName -v
```

## Architecture

### Three-Tier Stack

1. **Redis Operator** — reconciles `DbaasRedisAdapter` CRD; supports DBaaS, direct Redis, and Disaster Recovery modes
2. **Redis Monitoring Agent** — auto-generates `telegraf.conf`, exposes Prometheus metrics
3. **Redis Tests** — RobotFramework suite for integration/availability validation

### Operator Internal Structure

```
redis-operator/
├── api/v1/              # Legacy API version
├── api/v2/              # Current storage version (storageversion marker)
│   └── impl/            # Service builders per concern
│       ├── adapter/     # DBaaS adapter provisioning
│       ├── redis/       # Non-DBaaS Redis deployment
│       ├── monitoring/  # Monitoring agent deployment
│       └── robotTests/  # Robot test deployment
├── controllers/         # Kubernetes reconciler (DbaasRedisAdapterReconciler)
├── common/              # TLS, certificate management, Redis env setup
└── dbaas/pkg/
    ├── core/            # Constants
    ├── entity/          # DTOs (DbCreateRequestSettings, ConnectionProperties, TelegrafData)
    ├── redis/           # Redis client abstraction + mocks
    ├── services/        # DB lifecycle administration
    └── templates/       # Kubernetes manifest templates
```

### Reconciliation Flow

- `DbaasRedisAdapterReconciler` → delegates to `nosqldb-operator-core` shared library
- `RedisServiceBuilder` orchestrates sub-builders (adapter, monitoring, robotTests, redis)
- Implements `core.CommonReconciler` interface from shared library

### Deployment Modes

- **DBaaS** (default) — Redis provisioned on-demand via DBaaS aggregator; operator manages metadata + monitoring only
- **Non-HA** — single Redis instance deployed directly
- **DR (Disaster Recovery)** — active/passive replication via Cluster Replicator; configured via `disasterRecovery` spec section

### API Versions

`api/v2/` is storage version. `api/v1/` exists for conversion webhooks. After changing types in `api/v2/`, always run `make manifests && make generate`.

## Key Dependencies

- `controller-runtime v0.22.4` — operator framework
- `k8s.io v0.34.1` — Kubernetes client
- `qubership-nosqldb-operator-core` — shared reconciler library (external)
- `qubership-dbaas-adapter-core` — DBaaS integration (external)
- `go-redis` — Redis client
- `fiber` — HTTP framework (adapter endpoints)
- `zap` — structured logging

## Helm Chart

Located at `redis-operator/charts/helm/redis-operator/`. Two CRD variants:
- `crds/k8s_1.22_crd.yaml` — Kubernetes 1.22+
- `legacy_crd/crd.yaml` — older clusters

Resource profiles: `small` (dev/PoC), `medium`, `large`.

## Ports

| Port | Purpose |
|------|---------|
| 8070 | Webhook server |
| 6060 | pprof profiling |
| metrics | Disabled by default |

## Build & Test Commands

All primary targets run from `redis-operator/`:

```bash
# Code generation (run after changing types in api/)
make manifests   # Regenerate CRD and RBAC configs
make generate    # Regenerate DeepCopy methods

# Code quality
make fmt         # go fmt
make vet         # go vet

# Build
make build       # Build manager binary

# Tests
make test        # Run unit tests (requires envtest)
```

---
> Source: [Netcracker/qubership-redis](https://github.com/Netcracker/qubership-redis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
