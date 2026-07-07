## foundation

> > Universal instruction file for AI coding assistants (Claude, Codex, Cursor, Copilot, Windsurf, etc.)

# Foundation Intelligence

> Universal instruction file for AI coding assistants (Claude, Codex, Cursor, Copilot, Windsurf, etc.)

**New here?** Start with [`README.md`](README.md) → [`docs/PHILOSOPHY.md`](docs/PHILOSOPHY.md) → [`docs/foundation_quick_start.md`](docs/foundation_quick_start.md), then come back to this file for the agent operating contract.

## Terminology

| Term | Definition |
| :--- | :--- |
| **Foundation Core** | The shared infrastructure repository containing `server-kit`, `runtime-transport`, `runtime-sdk`, etc. |
| **Foundation Template** | The skeletal structure in `templates/` used to bootstrap new projects. |
| **Foundation Project** | A specific application (e.g., `trader_os`, `fintech_v1`) generated from the template. |
| **Foundation Reference** | The `/foundation` directory inside a **Project**, which is a local copy/reference to Core modules. |

## Project Context

This is a **Foundation** project - a production-grade full-stack application using Go backend, TypeScript/React frontend, and Rust/WASM for high-performance compute. The foundation provides shared infrastructure for event-driven, tenant-isolated, realtime applications.

## Agent Operating Baseline

Before editing architecture-sensitive code, read these files in order:

1. `docs/foundation_glossary.md` or `docs/foundation/foundation_glossary.md` — concept lookup and agent Q&A
2. `docs/foundation_quick_start.md` or `docs/foundation/foundation_quick_start.md`
3. `docs/foundation_tour.md` or `docs/foundation/foundation_tour.md`
4. `docs/foundation_architecture_contract.md` or `docs/foundation/foundation_architecture_contract.md`
5. `docs/agent_operating_contract.md` or `docs/foundation/agent_operating_contract.md`
6. `docs/practice_controls.md` or `docs/foundation/practice_controls.md`
7. `docs/ai_threat_model.md` or `docs/foundation/ai_threat_model.md` when tool, model, retrieved, generated, package, or security-sensitive input affects the change
8. The relevant practice file for the lane you are changing
9. `docs/future_practices_research.md` or `docs/foundation/future_practices_research.md` when proposing a new practice, security posture, performance lane, or agent workflow

Definition of Done for agent-authored changes:

1. State whether a public contract changed.
2. Identify the invariant that must still hold.
3. Leave evidence: test, benchmark, static check, review note, or migration proof.
4. Preserve or document the fallback path.
5. Name the scope boundary touched.
6. Add or update a regression guard.
7. Update docs or explain why no documentation changed.

## Tech Stack (2026 Standards)

| Layer | Technology | Version |
| ------- | ------------ | --------- |
| Backend | Go | 1.26+ |
| Frontend | TypeScript, React | 5.9+, 19.2+ |
| High-Performance | Rust, WASM | 1.95+ |
| Database | PostgreSQL | 18+ |
| Cache/Pubsub | Redis | 8+ |
| Queue | River (Go) | Latest |
| Protocol | Protocol Buffers, Cap'n Proto | 3.x |

## Architecture Overview

A **Foundation Project** is structured to separate shared infrastructure from application logic:

```text
project/
├── cmd/                    # Entry points (server, worker)
├── internal/               # Application logic
│   ├── service/           # Domain services (e.g., service/order, service/user)
│   ├── worker/            # Background job handlers
│   └── startup/           # App-specific initialization
├── api/                    # Protocol definitions
│   ├── protos/            # App-specific Protobuf schemas
│   └── schemas/           # Cap'n Proto schemas
├── frontend/              # React SPA
│   └── src/
│       ├── features/      # Feature modules
│       ├── components/    # Shared UI (wrapping foundation primitives)
│       ├── runtime/       # WASM bridge, transport adapters
│       └── stores/        # State management (Zustand + transport)
├── docs/
│   └── foundation/        # Foundation practices (copied from Core)
├── foundation/            # Shared infrastructure (READ-ONLY REFERENCE)
│   ├── server-kit/go/     # Backend modules
│   ├── runtime-transport/ # Client transport
│   ├── runtime-sdk/       # WASM kernel
│   ├── ui-minimal/        # UI Primitives
│   └── frontend-kit/      # Frontend operational utilities
└── rust/                  # Native Rust crates
```

## Critical Rules (Mandatory)

### 1. Foundation Dependency Boundary

**NEVER** import or alias raw source files from `foundation/*/ts/src` or internal Go packages.

- **Frontend**: Consume foundation logic via package boundaries: `@ovasabi/runtime-transport`, `@ovasabi/frontend-kit`, `@ovasabi/ui-minimal`.
- **Backend**: Use the `server-kit` module exports.

### 2. Correlation ID Propagation

Every mutating command MUST carry a `correlationId`. Trace it through all workers, events, and logs.

```go
// CORRECT
ctx = metadata.WithCorrelationID(ctx, envelope.CorrelationID)
bus.Publish(ctx, "domain:action:requested", payload)
```

### 3. Event Contract Lifecycle

All domain events follow `<domain>:<action>:<state>` pattern:

- `:requested` - Command received, validation passed
- `:success` - Operation completed
- `:failed` - Operation failed with reason

### 4. Tenant Isolation

Never trust client-supplied `organization_id`. Always derive from authenticated context via `auth.OrgIDFromContext(ctx)`.

### 5. Error Handling

Use the foundation error taxonomy (`foundation/server-kit/go/errors`). Never panic in request handlers.

### 6. Bounded Operations

All loops, retries, and external calls MUST have explicit bounds (timeouts, max attempts).

### 7. Just-in-Time Context Retrieval (Context Budgets)

To avoid context window overload, do NOT read all documentation files in bulk. Always start by reading `docs/foundation/foundation_glossary.md` to get the high-level concept index. Only load specific documentation files (like `transfer_lane.md` or `hermes_projection.md`) on-demand when modifying files that touch those specific domains.

### 8. Scaffold Ownership Verification

Before editing any file in the workspace, you MUST cross-reference `templates/scaffold.manifest.tsv` to check its sync mode. If a file is marked `overwrite` or `force`, it is owned by the Foundation core and must not be edited in project space, as local edits will be wiped on the next fleet update.

### 9. Evidence & Validation Tiers

Apply the appropriate severity validation tier to prevent over-engineering:

- **Tier 1 (Core & Critical)**: Touches money/currency math (`money/`), authentication, authorization (`auth/`, `policy/`), database schema/isolation, or WASM memory buffers. Requires full benchmarks, specs, regression tests, or formal proofs.
- **Tier 2 (Domain & Logic)**: Touches domain services, workers, or API handlers. Requires standard unit tests and static lints.
- **Tier 3 (Presentation & Copy)**: Touches CSS layout styles, copywriting, translation tokens, or non-functional UI code. Requires local lint verification only.

## Commands

```bash
# Build & Generation
make generate-contracts      # Full code gen (Protos -> Go/TS)
make build                   # Build all binaries
make frontend-build          # Build frontend production bundle

# Testing & Linting
make test                    # Run all tests (Go, TS, Rust)
make lint                    # Run all linters
make verify                  # Full CI verification suite

# Infrastructure
make docker-up               # Start dev stack (PG, Redis, etc.)
make migrate-up              # Run DB migrations
```

## Server-Kit Modules (Key Primitives)

| Module | Purpose |
| :--- | :--- |
| `errors` | Categorized error taxonomy with HTTP mapping. |
| `metadata` | Context-aware metadata (CorrelationID, TenantID, RequestID). |
| `auth` | RBAC and JWT context helpers. |
| `policy` | Cedar-inspired policy-as-code authorization. |
| `retry` | Standardized retry policies with exponential backoff + jitter. |
| `circuitbreaker` | Fault tolerance for external service calls. |
| `featureflags` | Targeted rollouts and environment-based overrides. |
| `tracing` | OpenTelemetry integration for distributed tracing. |
| `cache` | Cache-aside patterns with tag-based invalidation. |
| `healthcheck` | Liveness/Readiness probes for all dependencies. |
| `worker` | River-based background job handling. |
| `transfer` | Progress-bearing upload/download lifecycle with bookend events. |
| `projectiongw` | HTTP projection gateway for Hermes-backed read paths. |
| `bulk` | Bounded large-payload and resumable multipart operations. |
| `objectstore` | Object-storage helpers with tenant-scoped key derivation. |
| `intelligence` | Registry-level intelligence signal extraction. |
| `money` | Integer minor-unit financial arithmetic with checked operations. |
| `kernellane` | Native Rust/FFI/SHM compute lane dispatch and descriptor management. |
| `chain` | Worker chain helpers for bounded multi-step job composition. |
| `hermes` | Bounded projection reads with freshness and rebuild contracts. |
| `hermessnapshot` | Objectstore-backed durable projection snapshots for warm-from-snapshot. |
| `eventlog` | Append-only lifecycle evidence for traces and inspection. |
| `degradation` | Health monitoring with automatic fallback behaviors. |
| `versioning` | Header/path/query API versioning with deprecation support. |
| `resilience` | Coordinated health, circuit, retry, and degradation across dependencies. |
| `connector` | HTTP and WebSocket connectors with health probing and streaming. |

## Go SIMD Posture

Go 1.26 includes experimental `simd/archsimd` support behind `GOEXPERIMENT=simd`. Treat this as a measured, architecture-specific optimization lane:

- Do not expose `archsimd` vector types in public APIs.
- Keep scalar Go and Rust/WASM/FFI fallbacks for every SIMD path.
- Use only for bounded, benchmark-proven loops over contiguous numeric or byte data.
- Keep it out of tenant/auth/orchestration code and request handlers unless a benchmark proves the boundary cost is worth it.
- Gate CI/build use explicitly with `GOEXPERIMENT=simd`; ordinary builds must remain portable.

## Frontend & High-Performance

### Transport Layer

Use `@ovasabi/runtime-transport` for all backend communication. It handles binary envelopes, WebSocket routing, and automatic request deduplication.

### UI Primitives (`ui-minimal`)

Check `foundation/ui-minimal` before creating local components. Use `MinimalButton`, `MinimalInput`, `MinimalAppShell`, etc. App-level components should be thin wrappers.

### Runtime SDK (`runtime-sdk`)

The bridge for Rust/WASM execution. It uses a 4KB control-buffer contract for high-performance communication between the JS event loop and the WASM guest.

## Communicative Connections

The Foundation modules are linked through a unified "Nervous System":

1. **Envelopes**: Every message (HTTP, WS, Redis) is wrapped in a `RuntimeEnvelope` (`runtime-transport`).
2. **Resilience Runtime**: The `server-kit/go/resilience` module coordinates health, circuits, and degradation across all backend modules.
3. **WASM Kernel**: `runtime-sdk` provides the 4KB high-speed buffer for JS/Rust communication.
4. **Config Sync**: `config-contracts` ensures frontend and backend use the same validated schema.

## Development Context

| Context | Purpose | Key Commands |
| :--- | :--- | :--- |
| **Core Dev** | Developing the Foundation modules themselves in this repo. | `make test`, `make lint` |
| **Template Dev** | Updating the blueprints in `templates/` and `scaffold.manifest.tsv`. | `make lint-foundation` |
| **Project Dev** | Building a specific app *using* a generated scaffold. | `make dev`, `make foundation-update` |

## Gotchas and Anti-Patterns

1. **Don't ignore context deadlines**: Always pass `ctx` to DB and external calls.
2. **Don't use MutationObserver for business state**: Use Zustand stores.
3. **Don't store secrets in frontend**: Use environment-injected config or secure storage.
4. **Don't skip error wrapping**: Use `errors.Wrap(err, "context")` to preserve stack traces.

## Testing Requirements

- **Unit tests**: ≥95% statement coverage (`go test -cover`) for new/changed production code; legacy modules must improve toward 95% when touched and cannot regress without an approved exception. Enforced by `make check-coverage-ratchet` against per-package floors in `tooling/coverage_baseline.psv`.
- **Integration**: Mandatory for event contracts and critical flows.
- **E2E**: Required for auth guards and core user journeys.

## Coding Practice Rules (CP-*)

The foundation enforces 36 coding practices. Key ones to remember:

| Rule | Summary |
| ------ | --------- |
| CP-01 | No goto, no uncontrolled recursion |
| CP-02 | All loops/retries must be bounded |
| CP-03 | Functions ≤80 lines, complexity ≤15 |
| CP-04 | Never ignore error returns |
| CP-06 | Minimize mutable shared state |
| CP-10 | Events must be idempotent |
| CP-11 | All behavior needs tests |
| CP-16 | Adaptive concurrency, not fixed throttling |
| CP-18 | Rate limit all ingress APIs |
| CP-20 | Server-side authorization on every object |
| CP-36 | Agent-authored changes must carry evidence |

Full reference: `docs/foundation/coding_practices.md` in generated apps, or `docs/coding_practices.md` in the foundation source repo.

## File References

For deeper context, read these files (`docs/foundation/` in generated apps, `docs/` in the foundation source repo):

| Topic | File |
| ------- | ------ |
| **Glossary and agent Q&A** | `docs/foundation/foundation_glossary.md` or `docs/foundation_glossary.md` |
| Agent operating contract | `docs/foundation/agent_operating_contract.md` or `docs/agent_operating_contract.md` |
| Practice controls matrix | `docs/foundation/practice_controls.md` or `docs/practice_controls.md` |
| AI and agent threat model | `docs/foundation/ai_threat_model.md` or `docs/ai_threat_model.md` |
| Future practices research | `docs/foundation/future_practices_research.md` or `docs/future_practices_research.md` |
| Projection freshness | `docs/foundation/projection_freshness_contract.md` or `docs/projection_freshness_contract.md` |
| Low-level performance lab | `docs/foundation/performance_lab.md` or `docs/performance_lab.md` |
| Coding rules | `docs/foundation/coding_practices.md` or `docs/coding_practices.md` |
| Database patterns | `docs/foundation/database_practices.md` or `docs/database_practices.md` |
| Redis patterns | `docs/foundation/redis_practices.md` or `docs/redis_practices.md` |
| Migration rules | `docs/foundation/migration_practices.md` or `docs/migration_practices.md` |
| Performance | `docs/foundation/optimization_points.md` or `docs/optimization_points.md` |
| Delivery metrics | `docs/foundation/delivery_metrics_practices.md` or `docs/delivery_metrics_practices.md` |
| Runtime architecture | `docs/foundation/runtime_foundation.md` or `docs/runtime_foundation.md` |
| Foundation guide | `docs/foundation/foundation_guide.md` or `docs/foundation_guide.md` |
| Styling and motion | `docs/foundation/styling_design_practices.md` or `docs/styling_design_practices.md` |
| Animation references | `docs/foundation/references/README.md` or `docs/references/README.md` |
| Transfer lane | `docs/foundation/transfer_lane.md` or `docs/transfer_lane.md` |
| Adding a Rust performance unit | `docs/foundation/rust_unit_guide.md` or `docs/rust_unit_guide.md` |
| Frontend command registry | `docs/foundation/frontend_command_registry.md` or `docs/frontend_command_registry.md` |

## Security Checklist

Before merging any PR, verify:

- [ ] No secrets in code or logs
- [ ] Input validation at all boundaries
- [ ] Authorization checks on target objects (not just routes)
- [ ] Rate limiting on exposed endpoints
- [ ] CORS configured with explicit origins
- [ ] Correlation IDs propagated for audit trails
- [ ] AI/tool/retrieved content treated as untrusted input until validated
- [ ] Agent evidence ledger is present for security, persistence, runtime, or scaffold changes

---
Version: 0.0.1 | Last updated: 2026-07-04

---
> Source: [nmxmxh/foundation](https://github.com/nmxmxh/foundation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
