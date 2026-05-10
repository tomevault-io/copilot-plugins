## k8s4claw

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

k8s4claw is a Kubernetes operator for managing heterogeneous AI agent runtimes ("Claws"). One `Claw` CRD manages the full lifecycle: StatefulSet, Service, ConfigMap, ServiceAccount, PDB, PVCs, NetworkPolicy, Ingress, RBAC, IPC Bus sidecars, and channel sidecars.

Module: `github.com/Prismer-AI/k8s4claw` — Go 1.25, controller-runtime v0.23.1.

## Build & Test Commands

```bash
make build                # Build operator binary → bin/operator
make build-ipcbus         # Build IPC Bus binary → bin/ipcbus
make test                 # Run all tests with -race (needs setup-envtest)
make lint                 # golangci-lint run ./...
make vet                  # go vet ./...
make fmt                  # gofmt + goimports
make generate             # Generate deepcopy methods
make manifests            # Generate CRD YAML → config/crd/bases/
```

### Running a single test

Controller tests require envtest assets:

```bash
KUBEBUILDER_ASSETS="$(go run sigs.k8s.io/controller-runtime/tools/setup-envtest@latest use 1.32.x -p path 2>/dev/null)" \
  go test -race -run TestName -v -timeout 120s ./internal/controller/
```

Non-controller tests run normally: `go test -race ./internal/ipcbus/...`

### Pre-commit checklist

```bash
go vet ./...
golangci-lint run ./...
```

## Architecture

### CRDs (api/v1alpha1/)

- **Claw** — Primary resource. Immutable `runtime` field. Manages all sub-resources via a single `ClawReconciler`.
- **ClawChannel** — Communication channel definition (webhook, slack, etc). Tracked via `ReferenceCount`.
- **ClawSelfConfig** — Agent self-configuration requests with allowlist validation.

Shared types (RuntimeType, ChannelMode, CredentialSpec, PersistenceSpec, SecuritySpec, IngressSpec) live in `common_types.go`.

### Controllers (internal/controller/)

**ClawReconciler** (`claw_controller.go`) — Main orchestrator. Reconciles ~12 sub-resources per Claw:
- Sub-resource logic split across: `claw_statefulset.go`, `claw_pvc.go`, `claw_rbac.go`, `claw_ingress.go`, `claw_ipcbus.go`, `claw_channel_sidecar.go`, `claw_network_policy.go`, `claw_metrics_port.go`
- Finalizer: `claw.prismer.ai/cleanup` — handles PVC reclaim (Delete vs Retain) and channel ref-count
- Status: Phase transitions (Pending → Provisioning → Running → Degraded/Failed)

**ClawChannelReconciler** (`channel_controller.go`) — Counts referencing Claws via field indexer, updates status.

**ClawSelfConfigReconciler** (`selfconfig_controller.go`) — Validates requested actions against Claw's `allowedActions`, sets phase to Applied/Denied.

**AutoUpdateReconciler** (`autoupdate_controller.go`) — Polls OCI registry for new tags, applies rolling updates with circuit breaker.

### Runtime Adapter Pattern (internal/runtime/)

`RuntimeAdapter` interface (builder + validator) per runtime. Each adapter provides:
- Container image, probes, resource defaults, environment variables
- Gateway port, graceful shutdown period, config merge strategy

Registry populated at startup: OpenClaw(18900), NanoClaw(19000), ZeroClaw(3000), PicoClaw(8080), IronClaw(3001).

`pod_builder.go` constructs the full StatefulSet pod template from the adapter + Claw spec.

### IPC Bus (internal/ipcbus/)

In-pod message router between channel sidecars and runtime container:
- **Protocol bridges**: WebSocket (OpenClaw), TCP (PicoClaw), UDS (NanoClaw), SSE (ZeroClaw)
- **Reliability**: WAL (at-least-once), DLQ (BoltDB), Ring Buffer (backpressure)
- Runs as native sidecar (init container with `restartPolicy: Always`)
- All containers share `/var/run/claw` via `ipc-socket` emptyDir volume

### Webhooks (internal/webhook/)

- **ClawValidator** — Immutable runtime field, credential exclusivity (secretRef XOR externalSecret), runtime-specific validation
- **ClawDefaulter** — Sets `reclaimPolicy: Retain` when persistence is specified without explicit policy

### Test Infrastructure

`suite_test.go` sets up a shared envtest environment with all controllers and webhooks registered. Tests use `waitForCondition()` polling and unique namespaces per test for isolation.

Envtest limitations to be aware of:
- No garbage collector — ownerReference cascading deletion doesn't work
- No StatefulSet controller — pods/replicas don't actually start
- Status stays at Provisioning (never Running)
- PVC protection finalizer controller doesn't run

## Key Conventions

- Errors wrapped with `fmt.Errorf("failed to X: %w", err)`
- Labels: `app.kubernetes.io/name=claw`, `app.kubernetes.io/instance=<name>`, `claw.prismer.ai/runtime=<type>`, `claw.prismer.ai/instance=<name>`
- Sub-resource names: `<claw-name>` (STS, Service, SA, PDB), `<claw-name>-config` (ConfigMap), `<claw-name>-netpol` (NetworkPolicy)
- Channel sidecar containers named `channel-<channel-name>` in init containers with `restartPolicy: Always`

---
> Source: [Prismer-AI/k8s4claw](https://github.com/Prismer-AI/k8s4claw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
