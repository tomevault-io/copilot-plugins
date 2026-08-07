## gateway

> This repository contains the Go control plane for Nantian Gateway. It watches Kubernetes Gateway API resources, translates them into internal routing state, serves admin APIs, and publishes runtime snapshots to data planes over gRPC/xDS.

# Gateway Repository Guide

## Repository Role

This repository contains the Go control plane for Nantian Gateway. It watches Kubernetes Gateway API resources, translates them into internal routing state, serves admin APIs, and publishes runtime snapshots to data planes over gRPC/xDS.

Do not use this repository for Rust data plane, Dashboard, Helm chart, Website, or Proto source-of-truth changes. Those live in sibling repositories.

## Git Workflow

The workspace root is not a Git repository. This component directory is its own Git repository.

Make changes in an isolated worktree under `~/.opencode/worktrees/`, not directly in the checked-out `gateway/` main checkout. Do not merge a worktree branch back to `main` until the user explicitly approves. Do not push `main` until the user explicitly asks after merge approval.

Root `docs/` files are workspace notes and do not need to be committed with gateway changes unless the user explicitly asks for archival handling.

## Commands

Run commands from the gateway repository root.

- `make build` builds all Go packages.
- `make test` runs `go test -count=1 -timeout 5m ./...`.
- `go test ./internal/translator` runs focused translator tests.
- `go test ./internal/controller` runs focused controller tests.
- `go test ./internal/admin` runs focused admin API tests.
- `make e2e-smoke` runs the Kind smoke test.
- `make conformance` creates a Kind cluster and runs Gateway API conformance tests.
- No local protobuf generation target is currently defined here; protobuf source and generation workflow live in the sibling Proto repository.

Use focused `go test ./path` checks while iterating, then run the broader relevant target before committing.

## Project Map

- `cmd/manager/` starts the controller manager and wires runtime services.
- `internal/controller/` watches Kubernetes resources and coordinates full or partial rebuilds.
- `internal/translator/` converts Gateway API resources, policies, services, workloads, and extension objects into internal IR snapshots.
- `internal/xds/` publishes snapshots and status over gRPC/xDS to data planes.
- `internal/admin/` serves operational, topology, metrics, and management APIs for the Dashboard and operators.
- `internal/gatewayapi/` contains Gateway API helper logic, validation, encoding, and supported feature declarations.
- `internal/ir/` defines the internal routing and runtime model shared by translator and gRPC publication code.
- `deploy/` contains Kubernetes manifests and overlays.
- `gen/` contains generated protobuf code.

## Generated Code

Do not edit generated files under `gen/` by hand. Change the source `.proto` definitions in the sibling Proto repository and bring generated output into this repository only when the source change and generation command are clear.

## Translator Maintenance

The translator package is the highest-risk package in this repository. When changing it:

- Preserve Gateway API semantics for parent refs, route attachment, listener validity, backend refs, filters, and status conditions.
- Preserve ReferenceGrant and namespace scoping rules for cross-namespace references.
- Preserve BackendTLSPolicy, BackendLBPolicy, session persistence, AIService, TokenPolicy, and WasmPlugin precedence rules.
- Prefer shared indexes and support-object loaders over ad hoc list scans.
- Keep full rebuild and partial rebuild behavior aligned.
- Add or update focused tests for route semantics, backend policy precedence, ReferenceGrant behavior, status summaries, partial rebuild paths, and IR shape changes.

### Translator Package Size

The `internal/translator/` package has been split into 8 sub-packages (2026-07-18). Root retains 13 files with the `Translator` struct, full/partial rebuild orchestration, support object loaders, and workload translation.

Sub-packages:
- `shared/` — indexes, helpers, limits, metrics, status summaries
- `backends/` — backend cluster, TLS, LB policy, session translation
- `routes/` — route and filter translation, timeout parsing
- `listeners/` — frontend validation, mesh service translation
- `policies/` — route attachment, gateway queries, Wasm/Token translation
- `routepolicy/` — RoutePolicy translation
- `aiservice/` — AIService translation
- `testutil/` — shared test helpers

## Documentation And Comments

Use English by default for documentation and code comments. Add localized text only when editing existing localized user-facing content.

## Naming Conventions

### Getters

This codebase follows the Go convention that getter methods do not use a `Get` prefix:

```go
// Correct — no Get prefix
func (c *Config) SyncPeriodDuration() time.Duration { ... }
func (s *SnapshotStore) Current() *Snapshot           { ... }
func (t *Translator) ControllerName() string          { ... }

// Avoid — Get prefix is non-idiomatic in Go
func (c *Config) GetSyncPeriodDuration() time.Duration { ... }
```

The `Get` prefix is reserved for methods that perform non-trivial work beyond returning a field value (e.g., I/O, computation, or error handling). When the method simply returns a stored or derived value, omit the prefix.

### Boolean Functions

Functions and methods returning `bool` should use an `Is`, `Has`, `Can`, `Should`, or `Does` prefix that makes the caller read as a natural English question:

```go
// Correct
func (c *Config) DashboardEnabled() bool     { ... }
func IsAuthConfigured(opts Options) bool     { ... }
func HasAnyManagedParent(...) bool           { ... }
func IsSameResourceKind(left, right string) bool  { ... }

// Avoid — returns bool but name doesn't indicate a boolean result
func authConfigured(opts Options) bool       { ... }
func anyManagedParent(...) bool              { ... }
func sameResourceKind(left, right string) bool    { ... }
```

## Experimental Packages (`internal/gatewayexp`)

The `internal/gatewayexp/` directory contains experimental Gateway API extension types that are not yet stable. Each package defines a CRD with an alpha API version:

| Package | CRD | API Group | API Version |
|---------|-----|-----------|-------------|
| `aiservice` | AIService | `gateway.nantian.dev` | `v1alpha1` |
| `backendlb` | BackendLBPolicy | `gateway.networking.k8s.io` | `v1alpha2` |
| `tokenpolicy` | TokenPolicy | `gateway.nantian.dev` | `v1alpha1` |
| `wasmplugin` | WasmPlugin | `gateway.nantian.dev` | `v1alpha1` |
| `routepolicy` | RoutePolicy | `gateway.nantian.dev` | `v1alpha1` |

The `v1alpha1` / `v1alpha2` suffix lives in the `GroupVersion.Version` string within each package — there are no version suffixes in Go package import paths. Package paths remain flat (e.g., `internal/gatewayexp/aiservice`).

### Version Stability Plan

When a package stabilizes to v1:

1. **If the API is unchanged**: update `GroupVersion.Version` to `"v1"`.
2. **If the API changed**: create a new canonical package under `internal/gatewayexp/<name>_v1/` with the `v1` GroupVersion, keep the alpha package for backward compatibility during a deprecation window, then remove the alpha package once all consumers have migrated.
3. Each package is promoted independently — no single flag-day for all five.

The `backendlb` package uses `v1alpha2` from `gateway.networking.k8s.io` (the upstream Gateway API group), following upstream stability. Promotion to `v1` depends on the upstream Gateway API specification.

## Package Naming Conventions

This codebase follows the [Go package naming conventions](https://go.dev/blog/package-names): lowercase, single-word names with no underscores or mixedCaps.

### Multi-Word Package Rename (COMPLETED 2026 Q2)

6 packages were renamed across ~205 files. All renames are complete on disk and verified with `go build ./...` + `go test ./...`. For historical reference, the renaming was:

| Old | New | Purpose |
|-----|-----|---------|
| `gwapi` | `gatewayapi` | Remove project-specific "gw" abbreviation |
| `gwexp` | `gatewayexp` | Remove project-specific "gw" abbreviation |
| `backendlb` | `backend` (under gwexp) | Idiomatic single word |
| `lbpolicy` | `loadbalancing` | Descriptive of function |
| `tlspolicy` | `backendtls` | Mirror CRD name, distinguish from frontend |
| `nodeinfo` | `noderegistry` | Describes function (node registration/status) |

## Acceptance

Every change needs a spec, plan, and strict acceptance criteria. Record exact verification commands and results before marking work complete.

For documentation-only changes in this repository, run at least:

- `go test ./internal/translator` when touching translator documentation.
- `make test` unless the plan explicitly scopes a smaller command and records why.
- A local README link/path check when rewriting `README.md`.
- `git diff --check origin/main...HEAD`.

For behavior changes, add focused tests first and then run all affected package checks.

---

## Pending Improvements & Audit Findings

Last audited: 2026-07-14
> **Sync with root TODO.md**: Item 3 (Package Rename) is complete. Items 1-2, 4-5 remain pending with detailed analysis below. Root TODO.md may mark related sub-tasks as done (e.g., DeepEqual skip for status batching), but the full improvements here have not been implemented.

### Item 1: Control Plane Memory Optimization (P2)

**Current State:**
- **pprof enabled**: Separate HTTP server at configurable `pprof.addr` (default `127.0.0.1:6060`). Exposes `/debug/pprof/` endpoints with optional bearer-token auth. Defined in `cmd/manager/pprof.go`, wired in `cmd/manager/app.go` as a managed component (graceful serve/shutdown).
- **No custom memory metrics**: No `nantian_gw_controlplane_mem_*` or Go `runtime.MemStats` metrics are registered. Only admin API request metrics exist (`AdminAPIRequestsTotal`, `AdminAPIRequestDurationSeconds`).
- **In-memory indexing structures**:
  - `internal/admin/detail_index.go`: `snapshotDetailIndex` holds hash maps for listeners (by name), backends (by namespace+name), and routes (by kind+namespace+name). Rebuilt from scratch on every snapshot version change.
  - `internal/admin/list_cache.go`: TTL-based response cache (default 1s) for resource lists and service catalogs to avoid repeated Kubernetes API calls.
  - `internal/infrastructure/route_indexes.go`: Kubernetes field indexes (GatewayClass by controllerName, Gateway by gatewayClassName, Route by service parents) used by the infrastructure reconciler. These are server-side indexes on the API server, not in-process.
  - `internal/ir/types.go`: Full `Snapshot` materialized in memory with all Listeners, HTTPRoutes, GRPCRoutes, StreamRoutes, Backends, Secrets, Workloads as in-process slices.
  - `internal/infrastructure/inspector.go`: Infrastructure report uses maps (`serviceIndex`, `sliceIndex`) for observed vs. expected comparison during inspections.
- **Memory threats identified**:
  - `snapshotDetailIndex` rebuilds all three lookup maps (listeners, backends, routes) per snapshot — old snapshots are GC'd but intermediate copies exist during rebuild.
  - List endpoints (`/v1/listeners`, `/v1/routes`, `/v1/backends`) return full unfiltered slice copies with no pagination enforcement at the snapshot layer.
  - No buffer pool or `sync.Pool` usage for the frequent slice allocations in `filterListeners`/`filterRoutes`/`filterBackends`.
  - `Snapshot.Clone()` (in `internal/ir/clone.go`) performs deep copies for xDS distribution but doesn't use arena allocators.

**Recommendations:**
1. **Add Go runtime memory metrics**: Expose `go_memstats_alloc_bytes`, `go_memstats_heap_inuse_bytes`, `go_memstats_gc_cpu_fraction` as Prometheus metrics to establish baseline and detect regressions.
2. **Add snapshot memory metrics**: Track `snapshot_size_bytes` (approximate JSON serialized size) and `detail_index_build_duration_seconds`.
3. **Profile snapshot lifecycle**: Use existing pprof endpoints to capture heap profiles during steady state and full-rebuild cycles.
4. **Consider `sync.Pool` for slice buffers** in admin filter functions that allocate new slices per request.
5. **Evaluate arena allocator** for `Snapshot` clone paths (requires Go 1.22+ `arena` package).

### Item 2: API Aggregation — Control Plane Aggregated Endpoint (P1)

**Current State:**
- `/v1/summary` exists as an aggregated overview: returns `Summary` struct with counts (listeners, routes, backends, secrets), node status distribution, listener health, and snapshot sync state. It computes route/backend/listener counts from the snapshot but doesn't return object details.
- `/v1/dashboard/capabilities` returns feature-flag toggles (AI overview, services, etc.).
- **Individual list endpoints** exist for each resource type:
  - `GET /v1/listeners` — returns full listener list with optional filter (name, protocol, hostname, attachedRoute, sort, pagination).
  - `GET /v1/listeners/{name}` — single listener detail.
  - `GET /v1/routes` — returns HTTP+GRPC+Stream routes with optional kind filter, sort, pagination.
  - `GET /v1/routes/{kind}/{namespace}/{name}` — single route detail.
  - `GET /v1/backends` — returns backends with pagination.
  - `GET /v1/backends/{namespace}/{name}` — single backend detail.
  - `GET /v1/nodes` — returns node list with pagination.
  - `GET /v1/nodes/{nodeId}` — single node detail.
  - `GET /v1/infrastructure` — infrastructure report with filtering (state, role, kind, namespace, name, sort, pagination).
  - `GET /v1/service-catalog` — service catalog with filtering.
- **No `?include=` parameter exists** on any endpoint. Each resource type requires a separate API call.
- **No composite/aggregated endpoint** that returns multiple resource types in a single response.

**What's Needed:**
1. **`GET /v1/gateways`** — list all managed gateways with optional `?include=routes,listeners,backends,summary` parameter.
2. **`include=summary` mode**: Return the existing `/v1/summary` data embedded in each gateway resource (gateway-level summary, not cluster-wide).
3. **`include=routes` mode**: Embed route lists filtered by gateway parentRef.
4. **`include=listeners` mode**: Embed listener config with status.
5. **`include=backends` mode**: Embed backend references used by this gateway's routes.

**Design considerations:**
- Gateway-to-route parentRef mapping already exists in translator (routes reference gateways via parentRefs). Need a reverse index for filtering.
- Per-gateway summary would be a new subset of the existing `Summary` struct scoped to one gateway.
- Response size could be large — needs pagination and gzip support.
- Consider `include=counts` as a lightweight option (just counts, not full objects).

**Implementation notes** (verified 2026-07-15):
- **Precedent for `?include=`**: `parseIncludeAllBackends(raw string) (bool, error)` already exists in `internal/admin/query_support.go:62` as a query parameter parser for backend inclusion. This function demonstrates the exact pattern needed for `?include=routes,listeners,backends`.
- **Natural starting point**: `handleSummary` in `internal/admin/server_overview.go:60` currently only returns `Summary` struct with counts. The `buildSummary` function (line 76) already walks all listeners — adding object-level detail under an `?include=` gate would be a minimal extension.
- **All individual list handlers** already accept `url.Values` from `r.URL.Query()`:
  - `filterListeners` (`query.go:24`) — handles `name`, `protocol`, `hostname`, `attachedRoute`, `sort`, pagination
  - `filterRoutes` (`query.go:69`) — handles `kind`, `sort`, pagination
  - Backend query (`backend_view.go`) — handles pagination
- **No new gRPC/storage layer needed**: All data is already in the in-memory `ir.Snapshot` that `handleSummary` receives from `s.store.Current()`.

### Item 3: Multi-Word Package Rename Audit (P1) — COMPLETE ✅

**Status: Complete (2026 Q2).** All 6 renames executed across ~205 files and verified:
- `gwapi` → `gatewayapi`, `gwexp` → `gatewayexp`, `backendlb` → `backend`
- `lbpolicy` → `loadbalancing`, `tlspolicy` → `backendtls`, `nodeinfo` → `noderegistry`

See "Package Naming Conventions" above for the summary table.

### Item 4: Status Update Batching (P2)

**Current State:**
- **Two trigger paths**: (a) Full batch reconciliation via `ReconcilerRunner` (periodic ticker + external triggers + settle-debounced), (b) Per-object controllers driven by Kubernetes watches with controller-runtime workqueues.
- **Evaluation IS batched**: `loadState()` fetches all Gateway API resources in one etcd read; `evaluateRoutes()`/`evaluateGateways()` evaluate everything from that snapshot.
- **Writes are NOT batched**: Each object gets an individual `client.Status().Patch(ctx, desired, client.MergeFrom(&current))` call. 100 HTTPRoutes = 100 separate Patch calls, 50 Gateways = 50 separate Patch calls. Writes are sequential (not parallel).
- **Within a single object**, all conditions ARE coalesced — Accepted, Programmed, ResolvedRefs all go into one Patch. There is one write per object, not one per condition.
- **DeepEqual skip**: `apiequality.Semantic.DeepEqual(current.Status, desired.Status)` checked before every write — unchanged objects skip the API call entirely. This is the most impactful existing optimization.
- **Existing rate-limit mechanisms**:
  - `ReconcilerRunner` settle delay: debounces rapid-fire `QueueRun()` calls into a single reconciliation (configurable `SettleDelay`).
  - Trigger channel (cap 1): multiple concurrent triggers coalesce into one signal.
  - Retry with exponential backoff for failed scopes (25% jitter, 5 min max).
  - Per-object controllers: `workqueue.MaxOfRateLimiter` with exponential failure backoff (200ms base, 30s max) and bucket rate limiter (10 QPS, 100 burst).
  - Conflict retry: `retry.RetryOnConflict(retry.DefaultRetry, ...)` wraps every Patch call (10-step exponential backoff, ~30s max).
- **No dedicated status update queue/buffer**: Status writes happen synchronously and inline during reconciliation. The trigger channel at the ReconcilerRunner level is only a coalescing signal, not a work queue.
- **Metrics** (`status_update_metrics.go`): `statusUpdateConflictsTotal`, `statusUpdateRetriesTotal`, `statusUpdateErrorsTotal` — no batching-specific metrics (no batch size, batch latency, objects-skipped-via-DeepEqual count).
- **All writes use server-side merge patch** (`client.MergeFrom`): Only changed fields are sent, which is already the most efficient single-object write strategy Kubernetes provides.

**Key files**: `internal/status/reconciler.go` (Reconcile orchestration), `internal/status/reconciler_collections.go` (batch iterate-all-objects loops), `internal/status/reconciler_gateway_status.go` (per-gateway Patch), `internal/status/reconciler_route_status.go` (per-route Patch), `internal/status/reconciler_policy_status.go` (per-policy Patch), `internal/status/status_update_metrics.go` (prometheus counters), `internal/status/controllers.go` (per-object controllers + workqueue), `internal/controller/leader_runner_queue.go` (settle delay, trigger coalescing), `internal/controller/leader_runner_timers.go` (exponential retry backoff).

**Recommendations:**
1. **Parallelize status writes** (low risk, immediate impact): Change the sequential `for` loops in `reconciler_collections.go` to use a worker pool (e.g., `errgroup` with `SetLimit`). Route and policy status writes are trivially parallelizable — they share no mutable state. Reduces end-to-end reconciliation latency from O(N) to O(N/parallelism).
2. **Add batch metrics**: `status_updates_written_total` (actual Patch calls), `status_updates_skipped_total` (DeepEqual skip count), `status_batch_duration_seconds` (histogram of batch write phase duration).
3. **Evaluate server-side apply** (`client.Apply` with `ForceOwnership`): Eliminates Get-before-Patch pattern, reducing API calls by 50% per object.
4. **Pre-serialize Patch payloads** (if parallelizing): Compute all patches in parallel, send sequentially — decouples compute from I/O.

### Item 5: Incremental xDS Updates (P2)

**Current State:**
- **Purely SotW (State of the World)**: Every config change pushes the full snapshot to all subscribed data planes. No incremental/delta mechanism exists.
- **Proto definition** (`proto/gateway/control/v1/control.proto`, line 113-114): `ConfigSnapshot` comment explicitly states "Every field is replaced wholesale on each push (full-state snapshot, not delta)".
- **gRPC service**: Single `ConfigurationDiscoveryService` with `StreamConfiguration` (bidirectional streaming) + `ReportStatus` (unary). **No `DeltaDiscoveryService`**, no `DeltaDiscoveryRequest`/`DeltaDiscoveryResponse` messages, no delta-variant RPCs anywhere.
- **Snapshot flow**: Translator builds IR `Snapshot` → `SnapshotStore.Publish()` fans out to subscribers (buffer size 1, coalescing) → `StreamConfiguration` handler receives full `*ir.Snapshot` → `protoCache.get()` → `buildProjectedProtoSnapshot()` (Clone + project + proto-convert) → gRPC Send → data plane ACKs/NACKs.
- **Snapshot ID**: SHA-256 content digest of entire IR state (JSON-marshaled, status-stripped). `Publish()` deduplicates by ID — if content hasn't changed, no push occurs.
- **Proto cache** (`snapshot_proto_cache.go`): Caches `*controlv1.ConfigSnapshot` proto objects per `(snapshot.ID, projectionProfile)` key. Uses `sync.RWMutex` for read path, `singleflight.Group` to deduplicate concurrent builds. Cache is invalidated on version change. **Only caches proto structs, not wire bytes** — gRPC re-encodes on every `Send()`.
- **No diff logic**: No comparison between old and new snapshots. The word "diff" appears only in infrastructure inspector code (comparing service specs), not in xDS delivery.
- **`subscriptions` field unused for filtering**: The `DiscoveryRequest.subscriptions` field is passed to `nodeinfo.Registry` for observability only — all subscribers receive all snapshots regardless of their declared subscriptions.
- **No Envoy-style typed discovery services** (LDS/RDS/CDS/EDS/SDS): Everything bundled into one monolithic `ConfigSnapshot` in one stream.
- **Per-stream state**: One bidirectional stream per nodeID. New stream for same nodeID supersedes the old one. ACK timeout monitoring terminates stale streams.
- **Snapshot cost** (typical scale from benchmarks): ~96 Listeners, ~600 HTTPRoutes, ~300 GRPCRoutes, ~300 StreamRoutes, ~600 BackendClusters (×4 endpoints each), ~96 Secrets (with PEM material), ~1,200 Workloads.

**Key files**: `internal/xds/server.go` (gRPC service registration, Server struct), `internal/xds/server_stream.go` (StreamConfiguration handler, subscribe, build, send, ACK/NACK), `internal/xds/server_send.go` (serialized send channel with timeout/supersede), `internal/xds/snapshot_proto_cache.go` (proto cache with singleflight dedup), `internal/xds/snapshot_projection.go` (Clone + feature-filter projection), `internal/xds/snapshot_proto.go` (full IR→proto conversion), `internal/xds/features.go` (projection profiles), `internal/xds/server_registry.go` (per-nodeID stream dedup), `internal/ir/store.go` (SnapshotStore pub-sub), `internal/ir/clone.go` (deep-copy for projection), `internal/ir/types.go` (Snapshot struct), `proto/gateway/control/v1/control.proto` (protobuf definitions).

**What Delta xDS Would Require:**
1. **Proto changes**: Add `DeltaDiscoveryService` with `DeltaStreamConfiguration` RPC; add `DeltaDiscoveryRequest`/`DeltaDiscoveryResponse` messages with `resource_names_subscribe`, `resource_names_unsubscribe`, `initial_resource_versions`, `type_url`, `resources`, `removed_resources`, and a `Resource` wrapper (`name`, `version`, `resource` as `google.protobuf.Any`).
2. **Per-resource versioning**: Each resource (listener, route, backend, etc.) needs its own version identifier. Currently only a single snapshot-level ID exists. Requires changes in the translator/IR layer.
3. **Diff engine**: Between old and new IR snapshots, compute which resources were added, removed, or changed. Could live in new `internal/xds/delta_diff.go`.
4. **Per-stream resource state**: Each stream must track what resources it has (`xDS state-of-the-world per client`) — a per-stream resource map keyed by `type_url` and resource name.
5. **Cache extension**: Extend `snapshotProtoCache` to cache per-resource proto objects (not just whole snapshots) for targeted sends.
6. **Gradual rollout**: Delta as an additional service alongside existing SotW. Both share the same `SnapshotStore` subscription.

**Effort estimate**: Large. Hardest pieces are per-resource versioning in IR/translator and per-stream state tracking. Proto changes and cache extension are relatively straightforward. Priority: start with proto definitions and per-resource versioning.

---

### Verification

```bash
go build ./...  # PASSES (no errors)
```

---
> Source: [nantian-gw/gateway](https://github.com/nantian-gw/gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
