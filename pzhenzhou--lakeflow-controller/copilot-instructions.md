## lakeflow-controller

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

`lakeflow-controller` (Go module `github.com/pzhenzhou/lakeflow-controller`) is a Kubebuilder
v4.6.0 Kubernetes operator for data-processing workflows. It exposes a declarative API
(`lakeflow.io/v1alpha1`) and translates it into **Argo Workflows**, **Argo Events**, and
**Spark Operator** (kubeflow) resources, with **Volcano** as the batch scheduler. The compiled
binary is `manager`.

Note: build artifacts, image names, RBAC, and deployment labels still use the legacy
`lakeflow-controller` name (e.g. the default `IMG`, the `lakeflow-controller-system` namespace). Treat
`lakeflow-controller` as the current project name; those references are historical.

One namespaced CRD:
- **LakeFlow** — a workflow: trigger config + a DAG of tasks (spark / python / bash executors).

## Common Commands

```bash
make build            # generate + fmt + vet, then build bin/manager
make run              # run the controller against ~/.kube/config
make test             # generate manifests + envtest unit tests (excludes /e2e, /deploy)
make test-e2e         # Kind-based e2e suite (test/e2e), creates+destroys a cluster
make lint             # golangci-lint (config: .golangci.yml, v2 schema)
make lint-fix         # golangci-lint --fix

# Code generation — run after editing api/ types or +kubebuilder markers:
make manifests        # regenerate CRDs/RBAC/webhooks under config/ (controller-gen)
make generate         # regenerate zz_generated.deepcopy.go
make generate-client  # typed client/listers/informers (hack/update-codegen.sh)
make verify-client    # CI check that generated client is current
```

Toolchain binaries (controller-gen, kustomize, setup-envtest, golangci-lint) are auto-installed
into `./bin` at pinned versions by the Makefile — do not `go install` them globally.

### Running a single test

```bash
go test ./pkg/adapter/ -run TestSparkConfigBuilder        # one test in one package
go test ./pkg/adapter/ -run TestGolden -update            # regenerate golden fixtures (see below)
```

Most unit tests are plain `go test` and need no cluster. Tests under `internal/controller`
(`suite_test.go`) and some in `pkg/status` use **envtest**, which requires the Kubernetes test
binaries — run them via `make test` (sets `KUBEBUILDER_ASSETS`) rather than bare `go test`.

### Golden tests

`pkg/adapter/golden_test.go` is a characterization safety net: it converts sample LakeFlows to
Argo/Spark manifests and diffs them against `pkg/adapter/testdata/golden/*.golden.yaml`. A fixed
clock (`goldenClock`, Unix 1700000000) keeps versioned template names deterministic. After an
intentional change to conversion output, regenerate and review the diff:
`go test ./pkg/adapter/ -run TestGolden -update`.

## Architecture

### Reconcile loop (`internal/controller/lakeflow_controller.go`)

`LakeFlowController.Reconcile` orchestrates dedicated managers in a fixed order; the heavy lifting
lives in `pkg/*`, not the controller:

1. **Observe** — `status.WorkflowStatusManager.ObserveExecution` reads the underlying Argo/Spark
   resources to derive current phase. Status is persisted once at the end via a `defer`.
2. **Validate** — `validation.Manager.Validate`; failures set a Validation condition and requeue.
3. **Reconcile** — `reconciler.LakeFlowResourceProcessor.Process` creates/updates the Argo
   resources, returning a `ReconcileResult` (Created/Updated/NoChange/Error).
4. **Apply outcome** — status manager records the result.
5. **Rerun** — `rerun.WorkflowRerunManager` handles annotation-driven **resubmit** (clone a fresh
   workflow) and **retry** (re-run failed tasks via Argo API); resubmit takes priority over retry.
   Done *after* reconcile so spec changes land in templates first.
6. **Control state** — `ApplyControlState` reconciles Active/Suspend/Stopped/Terminated.

Watches: owns Argo `Workflow`/`CronWorkflow`/`WorkflowTemplate`, plus extra `Watches` that map
Argo resource events back to the owning LakeFlow (by owner-ref, falling back to the
`lakeflow.io/name` label for CronWorkflow-spawned workflows). Custom predicates in
`lakeflow_controller_predicate*.go` gate requeues. **Leader election is disabled**; concurrency is
expected to be handled per-namespace by deployment topology, not a single elected leader.

`WATCH_NAMESPACES` (comma-separated env var) limits the cache to specific namespaces; empty/unset
watches all namespaces. This backs the "namespace-mode" per-namespace deployment pattern.

### Trigger types → resources (`pkg/reconciler`)

The trigger class in `LakeFlowSpec.WorkflowTrigger` determines which Argo resources are created:

- **Schedule** (cron) → `WorkflowTemplate` + `CronWorkflow`
- **Immediate** → `WorkflowTemplate` + `Workflow`
- **Dependency** → `WorkflowTemplate` **only**. Cross-workflow dependency evaluation and the actual
  Workflow creation are performed by an *external* `lake-watcher` service — the operator does not
  evaluate dependencies itself.

### Package map

- `api/v1alpha1` — CRD types. `lakeflow_consts.go` is the source of truth for phases, control
  states, executor types, and all `lakeflow.io/*` label/annotation keys. `zz_generated.deepcopy.go`
  is generated; `install/` registers the scheme for external client use.
- `pkg/adapter` — converts LakeFlow → Argo/Spark manifests. **Builders are pure**
  (`WorkflowConverter`, `InfraDescriber`): they *describe* desired objects and never touch the
  cluster — the reconciler applies them. This purity is what makes golden tests possible. Spark
  config layering (Iceberg/Hive/OSS), versioned template naming, PVC builders, and retry-policy
  construction live here.
- `pkg/reconciler` — `Process` plus delegate components `resourceProcessor` / `resourceComparator`
  / `resourceValidator`. Compares generation vs. observedGeneration to skip no-op updates.
- `pkg/status` — `WorkflowStatusManager` is stateful per reconcile loop (holds a deep copy of the
  original status to diff). Split into `reconcileStateHandler` (resource outcome → phase) and
  `controlStateHandler` (Active/Suspend/Stopped/Terminated, incl. Spark + sensor lifecycle).
- `pkg/rerun` — retry/resubmit logic, active-workflow guard, failed-task DAG filtering.
- `pkg/cloud` — vendor storage abstraction (`Provider`). Pure, side-effect-free, not surfaced in
  any CRD; currently only Aliyun. Describes desired `StorageClass`/`PVC` objects.
- `pkg/validation`, `pkg/metrics`, `pkg/common` (shared logger + utils). `pkg/trigger` is currently
  empty.

### Important conventions

- The `pkg/cloud` Provider and `pkg/adapter` builders/describers must stay **pure** — no cluster
  reads/writes. Applying objects is solely the reconciler's job. Preserve this seam; it underpins
  golden testing and idempotent apply.
- Cluster-scoped objects (StorageClasses) use get-or-adopt semantics and are **never**
  owner-referenced from a namespaced LakeFlow. The shared OSS PVC is intentionally not
  owner-referenced either, so it isn't garbage-collected while other workflows still use it.
- When changing `api/` types or `+kubebuilder:` markers, always run `make manifests generate`
  (and `make generate-client` if the typed client surface changed) and commit the regenerated
  output.

.

---
> Source: [pzhenzhou/lakeflow-controller](https://github.com/pzhenzhou/lakeflow-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
