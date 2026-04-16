## kernel

> Generates SQL strings from compiled types:

# CLAUDE.md — Ecrous Kernel

> This file is the complete architectural context for the Ecrous Kernel.
> Read it fully before making any code changes. Every section represents a deliberate decision.
> The codebase contains extensive `IMPL` comments that serve as implementation blueprints — read them before implementing anything.

---

## Plan Tracking

Before starting any multi-step work, create or update `docs/plans/{YYYY-MM-DD}-{slug}.md`.
Format:
- [ ] Step description
- [x] Completed step

After completing each step, update the checkbox. When resuming work (`claude --continue`), 
read the active plan file first to understand where we left off.

When a plan is fully complete, move it to `docs/plans/done/`.

---

## Project Identity

- **Project name:** Ecrous Kernel (part of the Ecrous platform)
- **Slogan:** *"Model once, derive everything"*
- **Repository:** `ecrous/kernel`
- **License:** AGPL-3.0 with output exception — generated artifacts (APIs, schemas, types, admin UIs) belong entirely to users. The engine itself is protected from proprietary forks. Dual licensing with commercial option available later.
- **Companion project:** Ecrous Studio (`ecrous/studio`) — visual model editor. Slogan: *"Your system, visually"*
- **Dev environment:** Nix flake with crane for Rust builds. `cargo-watch` and `rust-analyzer` in devShell.

---

## The Core Philosophy

Data models and their invariants are the single source of truth. Users define entities, fields, relationships, state machines, permissions, and invariants once — either through the Studio's visual editor or by writing YAML directly. The Kernel reads that model at startup and mechanically derives everything else: database schemas with constraints, REST API endpoints, TypeScript types, admin interfaces, migration plans.

**Key principle:** Correctness is enforced by construction across all layers, not by convention. If the model validates, the derived system is correct.

**Key invariant:** At request time, the kernel does ZERO interpretation of the model. Everything is pre-compiled into lookup tables and prepared statements. The only runtime costs are: HashMap lookups (~20ns), Postgres round-trips (~1-2ms), and JSON serialization (~10-50μs).

---

## Relationship to Studio

```
┌─────────────────────┐         model.yaml         ┌──────────────────┐
│                     │                             │                  │
│   Studio (Svelte)   │  ──── writes to disk ────▶  │  Kernel (Rust)   │
│                     │                             │                  │
│  Visual editor      │         hot reload          │  HTTP API server │
│  Canvas, panels     │  ◀──── watches file ─────   │  Postgres        │
│  Live preview       │                             │  Prepared SQL    │
│  Verification       │                             │                  │
└─────────────────────┘                             └──────────────────┘
```

- The Kernel does NOT need the Studio. You can write model YAML by hand.
- The Studio does NOT need the Kernel. It's a standalone editor.
- They connect through the **model file format** — that's the contract.
- Do NOT make the Kernel the backend for the Studio (circular dependency).

---

## Architecture Overview

```
YAML model file
      │
      ▼
  ┌──────────┐     ┌──────────────┐     ┌─────────────┐
  │  Parse   │ ──▶ │   Compile    │ ──▶ │   Verify    │
  │ (serde)  │     │ model→runtime│     │ cross-layer │
  └──────────┘     └──────────────┘     └─────────────┘
                         │
                         ▼
                  CompiledModel
                  ├─ EntityRegistry   (entity name → compiled entity)
                  ├─ RoutingTable     (method+path → handler)
                  ├─ PreparedSQL      (per entity, per operation)
                  └─ EventDispatcher  (event type → plugin handlers)
                         │
                         ▼
                   Kernel::serve()
                  (Axum HTTP server)
```

Three-stage pipeline: **Model** (what the user writes, YAML) → **Compiled** (optimized runtime structures) → **Served** (HTTP API backed by Postgres).

---

## Current State of the Code

**Everything is scaffolded, nothing is implemented.** Every method body is `todo!()`. The types and architecture are fully designed with extensive `IMPL` comments that serve as blueprints. The code compiles (type-checks) but does nothing at runtime.

### What exists:
- Complete type definitions for the model layer (6 layers of types)
- Complete type definitions for the compiled layer (runtime-optimized structures)
- Engine component shells with detailed pseudocode in comments
- Error type hierarchy (build-time and request-time errors)
- Kernel struct with ArcSwap-based hot reload architecture
- Nix flake dev environment

### What needs implementing (in order):
1. `ValidatorFactory::build()` — field type → validator closure
2. `SqlBuilder` methods — compiled types → SQL strings
3. `Compiler::compile()` — Model → CompiledModel
4. `Verifier::verify()` — structural checks (Level 1)
5. `Model::from_file()` / `Model::from_dir()` — YAML parsing
6. `Kernel::new()` — boot sequence
7. `Kernel::serve()` — Axum HTTP server with generic handlers
8. `Kernel::hot_reload()` — atomic model swap
9. `Migrator::diff()` — old model vs new model → migration plan

---

## Module Structure

```
src/
├── lib.rs                    ← Kernel struct, module tree, re-exports
│
├── model/                    ← What the user writes (YAML-serializable)
│   ├── mod.rs                ← Model struct (top-level container), from_file/from_dir
│   ├── system.rs             ← Layer 0: SystemConfig (tenancy, audit, timestamps, roles, events)
│   ├── types.rs              ← Layer 1: FieldType enum (18 variants), constraints, CustomTypeDef
│   ├── entity.rs             ← Layer 2-5: EntityDef, FieldDef, RelationshipDef, InvariantDef
│   ├── lifecycle.rs          ← Layer 3: LifecycleDef, TransitionDef, SyncRule, FieldStateRule
│   ├── access.rs             ← Layer 4: PermissionsDef, RoleGrant, FieldPermission
│   └── condition.rs          ← Shared: Condition, ConditionOp, ConditionValue, aggregates, functions
│
├── compiled/                 ← What the kernel executes (optimized for O(1) lookups)
│   ├── mod.rs                ← CompiledModel, CompiledSystemConfig, EventDispatcher, EventHandler trait
│   ├── entity.rs             ← CompiledEntity, CompiledField (with validator closures), CompiledRelationship
│   ├── lifecycle.rs          ← CompiledLifecycle (transition lookup table), CompiledTransition (with SQL)
│   ├── permissions.rs        ← CompiledPermissions (RLS clauses, field visibility, transition roles)
│   └── routing.rs            ← RoutingTable, RouteTarget, RequestContext
│
├── engine/                   ← The transformation pipeline
│   └── mod.rs                ← Compiler, Verifier, SqlBuilder, ValidatorFactory, Migrator
│                                (all with detailed IMPL pseudocode in comments)
│
└── error.rs                  ← KernelError, ParseError, CompileError, VerifyError, RequestError, ValidationError
```

---

## Model Layer — The 6 Layers

The model is organized into 6 layers. Simple entities use only layers 0-2. Complex entities use all 6.

### Layer 0: System (`model/system.rs`)
Global configuration affecting every entity:
- **Tenancy:** `isolated` | `shared` | `none`. Adds tenant FK column + implicit WHERE clause to every query.
- **Audit:** transition logging (`always`|`opt_in`|`never`), field change tracking, retention period.
- **Timestamps:** auto-add `created_at`/`updated_at` to every entity (default: true).
- **Actor tracking:** auto-add `created_by`/`updated_by` FK→users (default: true).
- **Soft delete:** DELETE → `SET deleted_at = now()`, every SELECT adds `AND deleted_at IS NULL` (default: true).
- **Events:** CloudEvents format, glob patterns for which events to emit (`entity.*`, `lifecycle.*`).
- **Roles:** global RBAC roles used across all entity permissions.

### Layer 1: Types (`model/types.rs`)
`FieldType` is the most important enum — 18 variants:
- **Primitives:** Text, Integer, Number, Boolean, DateTime, Date
- **Semantic:** Email, Url, Phone, Currency, Percentage, Slug, Uuid, RichText, Json
- **Compound:** Select (enum/dropdown), Lifecycle (state machine reference), Custom (user-defined type reference)
- **Media:** File, Image

Each variant carries deep meaning for derivation: Postgres type, CHECK constraints, TypeScript type, branded types, form widget, validator, aggregatability, filterable operations. One word in YAML (`type: email`) → six derivation targets.

**Custom types** (`CustomTypeDef`) compose a base type with additional constraints: `positive_currency = currency + min: 0.01`. They don't create new PG types — just the base type with tighter CHECKs.

### Layer 2: Schema (`model/entity.rs`)
`EntityDef` is the central modeling unit:
- **Fields** (`FieldDef`): type, required, unique, immutable, default, description, indexed
- **Relationships** (`RelationshipDef`): target entity, cardinality (many_to_one, one_to_many, one_to_one, many_to_many), cascade policy (cascade, restrict, set_null, detach), required
- Auto-fields injected by compiler: `id` (UUID), timestamps, tenant FK, soft_delete column, actor FKs

### Layer 3: Lifecycle (`model/lifecycle.rs`)
State machines — **the layer that separates this platform from every CRUD generator**.
- **States:** named set with optional metadata (description, color)
- **Transitions:** from state(s) → to state, with trigger type (manual/event/automatic), required fields, guard conditions, cross-lifecycle guards, hooks
- **Field-state rules:** field behavior changes per state (immutable, required, hidden, editable)
- **Sync rules:** cross-lifecycle coordination when one lifecycle enters a state, force transition on another lifecycle (within same DB transaction)
- Entities can have 0 (pure CRUD), 1 (typical), or 2+ (complex parallel state machines like payment + fulfillment) lifecycles

### Layer 4: Access (`model/access.rs`)
Three levels of permissions:
1. **CRUD permissions:** role → grant with optional row-level WHERE conditions (→ Postgres RLS)
2. **Field permissions:** role × field → hidden | read | read_write
3. **Transition permissions:** role → allowed transitions

### Layer 5: Invariants (`model/entity.rs` - `InvariantDef`)
Cross-field constraints that must always hold: `refund_amount <= total`. Enforced at API level (validator closure), database level (CHECK constraint), or form level (client-side validation).

### Shared: Conditions (`model/condition.rs`)
The universal expression language used by guards, invariants, permissions, and query filters. Structured (not a DSL string) so they can be compiled to SQL, TypeScript, Rust closures, and Z3 expressions.

Supports: field vs literal, field vs field reference (`{ref: total}`), field vs context reference (`{ctx: user_id}`), set membership, null checks, time-relative comparisons, relationship aggregates (`items.sum(quantity) <= 100`), field functions (`length(description) >= 100`).

Covers ~95% of real-world guards and invariants. Remaining 5% = escape hatch via plugins.

---

## Compiled Layer — Runtime-Optimized Structures

### CompiledModel (`compiled/mod.rs`)
Top-level container: entity registry (HashMap), routing table, event dispatcher, system config snapshot. Lives inside `ArcSwap` for hot reload.

Memory: ~20KB per entity, ~2MB for 100 entities. Tiny — Postgres connections are the real cost.

### CompiledEntity (`compiled/entity.rs`)
Everything needed to serve CRUD + lifecycle for one entity:
- Fields as Vec with HashMap index for O(1) name lookup
- **Validator closures** — pre-built at compile time with constraints captured. No type dispatch at runtime. `(field.validator)(value)?` is a direct function pointer call.
- Pre-built SQL strings for list, get, create, update, delete
- Compiled lifecycles, permissions, invariants

### CompiledLifecycle (`compiled/lifecycle.rs`)
The state machine as a lookup table: `HashMap<(current_state, transition_name), CompiledTransition>`.

**Performance:** transition = HashMap lookup (20ns) + role check (10ns) + required field validation (30ns) + execute SQL (1-2ms). Total Rust overhead: ~60ns.

Each `CompiledTransition` contains: target state, allowed roles (HashSet), required fields, field validators, **the SQL UPDATE statement with all guards baked into the WHERE clause**, audit INSERT SQL, hook event keys.

**The magic:** the ENTIRE transition logic is one SQL query. State check, guards, cross-guards, tenant filter, soft delete filter — all in the WHERE clause. If WHERE doesn't match → 0 rows affected → 409 Conflict. No read-then-write. One round-trip.

### CompiledPermissions (`compiled/permissions.rs`)
Pre-compiled lookup tables:
- Read RLS policies: `role → SQL WHERE clause string` (injected into SELECTs)
- Create roles: `HashSet<role>` (simple membership check)
- Update/delete policies: `role → SQL WHERE clause`
- Hidden fields per role: `role → HashSet<field_name>` (stripped from responses)
- Readonly fields per role: `role → HashSet<field_name>` (rejected in updates)
- Transition roles: `"{lifecycle}.{transition}" → HashSet<role>`

Total authorization cost per request: ~50ns.

### Routing (`compiled/routing.rs`)
Segment-based pattern matching, not a traditional router:
```
GET    /api/{entity}           → list
POST   /api/{entity}           → create
GET    /api/{entity}/{id}      → get
PATCH  /api/{entity}/{id}      → update
DELETE /api/{entity}/{id}      → delete
POST   /api/{entity}/{id}/{lifecycle}/{transition} → transition

GET    /meta/model             → serve raw model
GET    /meta/health            → health check
POST   /meta/reload            → trigger hot reload
POST   /meta/migrations/preview → migration diff
```

Entity name validated against `entities` HashMap. No Axum router for entity routes — routes are model-derived and swapped on hot reload.

`RequestContext` extracted from JWT: user_id, tenant_id, role, additional claims. This is what `{ctx: user_id}` in Conditions resolves to at runtime.

---

## Engine — The Transformation Pipeline

Four components + migrator, all in `engine/mod.rs`:

### 1. ValidatorFactory
Builds validator closures from field types. Each semantic type has a factory method. Constraints are captured at compile time into the closure. Detailed match-arm pseudocode in comments for Text, Email, Currency, Select, Boolean.

**Build first.** Needed by the Compiler for field validators.

### 2. SqlBuilder
Generates SQL strings from compiled types:
- **DDL:** CREATE TABLE, CREATE INDEX, ALTER TABLE (for migrations)
- **DML:** SELECT, INSERT, UPDATE, DELETE (for prepared statements)

Design: SQL as strings, not AST. Simpler, debuggable, built once at compile time.

Key method: `transition_query()` — the most important SQL in the system. Builds UPDATE with SET clause (new state + required fields + timestamps) and WHERE clause (id + current state + tenant + soft_delete + guards + cross-guards). All in one query.

**Build second.** Needed by the Compiler for prepared SQL.

### 3. Compiler
Main entry: `Compiler::compile(model: &Model) -> Result<CompiledModel, CompileError>`

Pipeline:
1. Resolve custom types (detect cycles, resolve chains)
2. Compile each entity (inject auto-fields, resolve types, build validators, compile lifecycles, compile permissions, compile invariants, generate SQL)
3. Validate cross-entity references (relationships point to existing entities, cross-lifecycle guards reference existing lifecycles)
4. Build routing table
5. Build event dispatcher (empty — plugins register later)
6. Build system config snapshot

~20ms for 50 entities.

### 4. Verifier
Three verification levels:
- **Level 1 (structural, ~5ms):** BFS reachability, dead-end states, role references, field references, relationship targets, duplicate names. **Implement first.**
- **Level 2 (Z3 constraint solving, ~200ms):** Guard satisfiability, invariant consistency, guard-invariant interactions, implied invariants. **Implement when Z3 is integrated.**
- **Level 3 (TLA+/Spin behavioral, ~1s):** Deadlock freedom, liveness, safety properties. **Implement last.**

### 5. Migrator
Diffs old vs new CompiledModel → MigrationPlan:
- **Safe changes** (auto-apply): add nullable field, add entity, add index
- **Destructive changes** (need confirmation): remove field, change type, remove entity, add required field without default

Hot reload flow: if destructive → return `Err(NeedsConfirmation(plan))`. User confirms → apply.

---

## The Kernel Struct

```rust
pub struct Kernel {
    compiled: ArcSwap<CompiledModel>,  // Atomically swappable for hot reload
    db: PgPool,                         // Postgres connection pool (persists across reloads)
    source: ArcSwap<Model>,            // Raw model for diffing and /meta/model endpoint
}
```

### Boot Sequence (`Kernel::new`)
1. Parse model from YAML
2. Connect to Postgres (`PgPoolOptions::new().max_connections(20).connect(url)`)
3. Compile model → CompiledModel
4. Verify cross-layer
5. Apply migrations if needed
6. Return Kernel with ArcSwap wrappers

Expected: ~85ms without Z3, ~300ms with.

### Serving (`Kernel::serve`)
Build Axum Router from compiled RoutingTable. Each handler receives `Arc<Kernel>` via Axum state, calls `self.compiled.load()` to get current model. Bind to addr, graceful shutdown on SIGTERM.

### Hot Reload (`Kernel::hot_reload`)
1. Parse new model
2. Compile → new CompiledModel
3. Verify
4. Diff old vs new → MigrationPlan
5. If destructive → `Err(NeedsConfirmation(plan))`
6. Apply safe migrations
7. `self.compiled.store(Arc::new(new_compiled))`
8. `self.source.store(Arc::new(new_model))`

In-flight requests keep their `Arc<CompiledModel>` reference to old model. New requests get new model. No locks, no downtime. Target: ~35ms reload.

---

## Event System

Async, non-blocking. `EventDispatcher` maps event type strings to `Vec<Arc<dyn EventHandler>>`.

Events dispatched via `tokio::spawn` — handlers run concurrently and NEVER block the HTTP response. Failed handlers are logged, not propagated. Zero impact on request latency.

Event types: `"entity.created"`, `"Order.created"`, `"Order.payment.refund.on_complete"`.

`EventPayload` contains: event type, entity name, entity ID, tenant ID, actor ID, full entity data as JSON, transition data (if lifecycle event), timestamp.

Plugins implement the `EventHandler` trait and register during compile phase.

---

## Dependencies

```toml
serde + serde_json + serde_yaml  # Serialization
tokio (full)                      # Async runtime
axum 0.8                          # HTTP (thinnest layer over hyper/tower)
tower + tower-http                # Middleware (CORS, tracing)
sqlx 0.8 (postgres, uuid, chrono, json)  # Postgres with runtime-prepared statements
jsonwebtoken 10                   # JWT auth
uuid + chrono                     # Core types
regex                             # Pattern validation
thiserror + anyhow                # Error handling
tracing + tracing-subscriber      # Logging
arc-swap                          # Atomic swap for hot reload
async-trait                       # Async trait methods (for EventHandler)
once_cell                         # Lazy statics for empty defaults
```

---

## Error Hierarchy

Two categories:

### Build-time (startup or hot reload)
- `ParseError`: invalid YAML, file not found, invalid structure
- `CompileError`: unknown type, unknown relationship target, guard field not found, undeclared state, unknown role, duplicates, required field with no default
- `VerifyError`: unreachable state, dead-end state, unsatisfiable guard (Z3), contradicting invariants (Z3), no role can transition

### Request-time (HTTP handling)
- `ValidationError` → 400: required missing, wrong type, below min, above max, too long, pattern mismatch, invalid option, immutable in state, invariant violation
- `Unauthorized` → 401: no/expired/invalid token
- `Forbidden` → 403: valid token but role lacks permission
- `NotFound` → 404: entity type not in registry, or row not found
- `Conflict` → 409: transition failed (wrong state, guard failed, concurrent update)
- `Internal` → 500: database error, serialization error

All validation errors are collected per-field and returned as a list (not just the first one).

---

## Model File Format

### Single file
```yaml
version: 1
system:
  name: "my-app"
  tenancy: { strategy: shared, tenant_entity: Organization, tenant_field: org_id }
  audit: { transitions: always, field_changes: opt_in, retention: "90 days" }
  timestamps: true
  actor_tracking: true
  soft_delete: true
  events: { format: cloudevents, emit: ["entity.*", "lifecycle.*"] }
  roles: [admin, user, support, warehouse, customer]
custom_types:
  positive_currency: { base: currency, min: 0.01 }
entities:
  Order:
    fields:
      email: { type: email, required: true }
      total: { type: currency, required: true, min: 0.01 }
    lifecycles:
      payment:
        initial: pending
        terminal: [captured, failed]
        states: { pending: {}, authorized: {}, captured: {}, failed: {}, refunded: {} }
        transitions:
          authorize: { from: pending, to: authorized }
          capture: { from: authorized, to: captured }
          refund:
            from: captured
            to: refunded
            requires: [refund_amount]
            guard:
              - { field: refund_amount, op: lte, value: { ref: total } }
    permissions:
      read:
        - { role: customer, where: [{ field: customer_id, op: eq, value: { ctx: user_id } }] }
        - { role: [admin, support] }
      create: { role: [customer, admin] }
    invariants:
      - name: "Refund <= Total"
        condition: { field: refund_amount, op: lte, value: { ref: total } }
        enforce: [api, database]
```

### Multi-file (also supported)
```
model/
  system.yaml        → SystemConfig
  types.yaml         → custom_types
  entities/
    order.yaml       → one EntityDef
    customer.yaml    → one EntityDef
```

The compiler doesn't care whether it came from one file or many. Always include `version: 1` — when format changes, write migrations from version N to N+1.

---

## Implementation Order (Recommended)

### Phase 1: Core Pipeline (make it compile and serve a basic API)
1. `Model::from_file()` — parse YAML into Model
2. `ValidatorFactory::build()` — implement for all 18 FieldType variants
3. `SqlBuilder` — DDL generation (CREATE TABLE) + DML generation (SELECT, INSERT, UPDATE, DELETE)
4. `Compiler::compile()` — follow the detailed pseudocode in engine/mod.rs
5. `Verifier::verify()` — Level 1 structural checks only
6. `Kernel::new()` — boot sequence
7. `Kernel::serve()` — Axum server with generic CRUD handlers
8. Test with hand-written YAML model files

### Phase 2: Lifecycle Transitions
1. Transition SQL generation (the magic single-query pattern)
2. Transition handler (POST /api/{entity}/{id}/{lifecycle}/{transition})
3. Cross-lifecycle guards
4. Sync rules (same-transaction forced transitions)
5. Field-state rules (immutable/required/hidden per state)

### Phase 3: Permissions & Auth
1. JWT extraction middleware
2. CRUD permission checks
3. RLS WHERE clause injection
4. Field-level visibility filtering
5. Transition role checks

### Phase 4: Hot Reload & Migration
1. `Migrator::diff()` — model comparison
2. `Kernel::hot_reload()` — atomic swap with ArcSwap
3. File watcher for dev mode (connect with Studio)
4. Migration preview endpoint (`POST /meta/migrations/preview`)

### Phase 5: Formal Verification (Future)
1. Z3 integration for guard satisfiability and invariant consistency
2. TLA+ for behavioral verification (deadlock, liveness)
3. WASM build of verifier for Studio integration

---

## Key Design Decisions

### Why ArcSwap instead of RwLock?
ArcSwap gives lock-free reads. Every request reads the compiled model — with RwLock, readers would contend. ArcSwap makes reads zero-cost (just an atomic load). Writes (hot reload) are rare and atomic.

### Why pre-built SQL strings instead of a query builder?
SQL strings are simpler, debuggable (you can log them), and built once at compile time. The string building cost is ~1-2μs at startup, never at request time. A SQL AST adds complexity for no runtime benefit.

### Why validator closures instead of runtime type dispatch?
`(field.validator)(value)?` is a direct function pointer call — no match statement, no type lookup. The closure was built at compile time with constraints baked in. This is measurably faster for hot-path validation.

### Why segment-based routing instead of Axum's router?
Entity routes are model-derived and change on hot reload. Rebuilding Axum's router on every hot reload is wasteful. Segment matching is O(1) and trivially swappable.

### Why Axum specifically?
Thinnest layer over hyper/tower. The kernel builds its own routing — Axum just dispatches HTTP. Minimal framework opinions, maximum control.

### Why all guards in the WHERE clause?
One SQL query handles the entire transition: state check + data guards + cross-lifecycle guards + tenant filter + soft delete. If any condition fails, 0 rows affected → 409 Conflict. No read-then-write pattern, no race conditions, one round-trip to Postgres.

---

## Code Style

- Extensive `IMPL` comments throughout the codebase serve as implementation blueprints. **Read them before implementing.**
- Comments use `///` for doc comments and `//` for implementation notes.
- Section separators: `// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━`
- Types are in `model/` (user-facing, serializable) and `compiled/` (runtime, optimized).
- Engine code lives in `engine/mod.rs` — consider splitting into submodules as implementations grow.
- All `serde` derives use `#[serde(rename_all = "snake_case")]` for enums.
- HashMap keys are strings (entity names, field names, state names, role names).
- Extensive use of `#[serde(default)]` for optional fields with sensible defaults.

---

## Testing Strategy

1. **Unit tests per engine component:** ValidatorFactory (test each type variant), SqlBuilder (test generated SQL strings), Compiler (test model → compiled roundtrip)
2. **Integration tests with Postgres:** Use sqlx test fixtures. Load a model YAML, compile, apply migrations, hit endpoints.
3. **Snapshot tests for SQL:** Assert that generated DDL/DML matches expected strings. Catch unintended SQL changes.
4. **The e-commerce demo model** (Order with dual lifecycles + Customer + Tag + OrderItem) should be the primary test fixture — it exercises every feature.

---

## What NOT to Build Yet

- Plugin system (design event hooks, don't build plugin loading)
- GraphQL API (REST first, GraphQL as a plugin)
- Admin UI generation (Studio handles preview; actual admin UI is a separate deliverable)
- Real-time subscriptions / WebSocket
- File/image upload handling (just store the reference URL)
- Query/analytics layer (reporting, aggregations, dashboards)
- Multi-region / distributed deployment
- Rate limiting, caching (these are deployment/infrastructure concerns)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ecrous) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
