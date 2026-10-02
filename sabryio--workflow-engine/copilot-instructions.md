## workflow-engine

> A durable workflow engine with a Rust core and network API. Combines event-driven fan-out with direct invocation as one unified surface.

# Workflow Engine

A durable workflow engine with a Rust core and network API. Combines event-driven fan-out with direct invocation as one unified surface.

## Architecture

Clean Architecture with explicit layer boundaries:

```
workflow-engine/
├── src/                          # Core library (Clean Architecture layers)
│   ├── domain/                   # Pure business logic (no dependencies on other layers)
│   │   ├── workflow.rs           # Workflow trait, ErasedWorkflow
│   │   ├── context.rs            # Context API
│   │   ├── event.rs              # Event trait
│   │   ├── realtime.rs           # RealtimeContext (domain API wrapper)
│   │   └── types/                # Domain value objects
│   │       ├── ids.rs            # RunId
│   │       ├── run.rs            # RunSnapshot, RunStatus, StepRecord
│   │       ├── error.rs          # WorkflowError
│   │       └── retry.rs          # RetryPolicy, WorkflowConfig
│   ├── ports/                    # Trait abstractions (dependency inversion)
│   │   └── traits.rs             # StateStore, Clock, EventPublisher, EventSubscriber,
│   │                             # RealtimeBus, WorkflowInvoker
│   ├── adapters/                 # Infrastructure implementations
│   │   └── memory/               # Built-in memory adapters
│   │       ├── state.rs          # MemoryStateStore
│   │       ├── event_bus.rs      # MemoryEventBus
│   │       ├── clock.rs          # RealClock, TestClock
│   │       └── realtime.rs       # BroadcastRealtimeBus
│   └── application/              # Use case orchestration
│       ├── orchestrator.rs       # Orchestrator + test utilities
│       ├── registry.rs           # Workflow registration
│       ├── admin.rs              # AdminClient
│       ├── remote.rs             # RemoteHub
│       └── remote_workflow.rs    # RemoteWorkflow
└── examples/
    └── http-server/              # HTTP API example (optional) — Clean Architecture
        ├── src/
        │   ├── bootstrap/        # Configuration & logging initialization
        │   │   ├── config.rs     # Config struct (host, port, log_level)
        │   │   ├── env.rs        # Environment defaults
        │   │   └── logging.rs    # Tracing subscriber setup
        │   ├── composition/      # Dependency wiring
        │   │   ├── app_state.rs  # AppState + FromRef impls
        │   │   └── builder.rs    # Dependency graph construction
        │   ├── dto/              # Data Transfer Objects (HTTP contracts)
        │   │   ├── error.rs      # ApiError, ApiWrap, error responses
        │   │   ├── workflow.rs   # Trigger/Schedule/Publish DTOs
        │   │   ├── run.rs        # Run management DTOs
        │   │   ├── worker.rs     # Worker registration DTOs
        │   │   ├── admin.rs      # Admin operation DTOs
        │   │   └── realtime.rs   # Realtime/SSE DTOs
        │   ├── handlers/         # HTTP request handlers (transport layer)
        │   │   ├── workflow.rs   # trigger, schedule, publish
        │   │   ├── run.rs        # get_run, list_runs, history, write_step, cancel, retry
        │   │   ├── worker.rs     # register_worker, unregister_worker, lease, complete
        │   │   ├── admin.rs      # pause, resume, drain, dead_letters
        │   │   ├── health.rs     # health, metrics
        │   │   ├── event.rs      # wait_event (long-poll)
        │   │   └── realtime.rs   # subscribe_channel, realtime_publish
        │   ├── server.rs         # Router assembly (pure function: AppState → Router)
        │   └── main.rs           # Entry point (bootstrap → composition → server → TCP)
```

**Key Point**: The core `workflow-engine` library has NO HTTP dependencies. HTTP is just one example of how to expose the engine. You can embed the orchestrator directly in your application, or build adapters for CLI, gRPC, message queues, Lambda, etc.

**Dependency flow**: domain ← ports ← adapters/application. Domain has zero dependencies on infrastructure or orchestration.

### Project Structure

Single library crate with Clean Architecture layers:

- **src/domain/** - Pure business logic: Workflow trait, Context API, domain types (RunId, WorkflowError, etc.)
- **src/ports/** - Trait abstractions for dependency inversion (StateStore, Clock, EventPublisher, etc.)
- **src/adapters/** - Infrastructure implementations (memory/, future: postgres/)
- **src/application/** - Use case orchestration (Orchestrator, Registry, AdminClient, RemoteHub)
- **examples/http-server/** - HTTP server exposing the engine using Clean Architecture:
  - **bootstrap/** - Configuration loading, logging setup, environment parsing
  - **composition/** - Dependency wiring (orchestrator, registry, hub, buses)
  - **dto/** - HTTP request/response types (API contracts)
  - **handlers/** - HTTP request handlers grouped by feature area
  - **server.rs** - Pure router assembly function
  - **main.rs** - Entry point orchestrating bootstrap → composition → server → TCP
- **tests/** - Inline in `src/application/orchestrator.rs` under `#[cfg(test)] pub mod test`

**TypeScript SDK** at `sdks/typescript/` - Client + worker runtime, package name `@workflow-engine/client`

## Setup

### Prerequisites

- Rust 1.75+ (edition 2021)
- Bun for TypeScript SDK

### First Time Setup

```bash
# Build the core library
cargo build

# Build TypeScript SDK
cd sdks/typescript
bun install
bun run build
```

## Development Commands

### Core Library

```bash
# Build library
cargo build

# Run library tests (uses TestClock, no HTTP needed)
cargo test --lib

# Run integration tests
cargo test

# Check for compile errors
cargo check

# Format code
cargo fmt --all

# Run clippy
cargo clippy --workspace
```

### HTTP Example Server

```bash
# Run the HTTP example (from workspace root)
cargo run -p http-server
# Listens on http://localhost:8080

# Check health endpoint
curl http://localhost:8080/health

# Build the example
cargo build --bin http-server
```

### TypeScript SDK

```bash
cd sdks/typescript

# Install dependencies
bun install

# Build (compiles to dist/)
bun run build

# Run tests
bun test

# Run full worker+client example
bun examples/worker-demo.ts

# Run client-only example (requires worker already running)
bun examples/trigger.ts
```

## Testing Strategy

### Rust Tests

- **Unit tests**: Inline with implementation (`#[cfg(test)] mod tests { }`)
- **Integration tests**: Inline in `src/orchestrator.rs` under `#[cfg(test)] pub mod test`
- **Test harness**: `orchestrator::test::test_client()` for in-process testing with virtual clock
- All tests must pass before merging: `cargo test --workspace`

### Key Test Utilities

```rust
use workflow_engine::application::orchestrator::test::test_client;

let (orchestrator, test_engine) = test_client(registry);
test_engine.advance_time(Duration::from_secs(3600)); // instant fast-forward
test_engine.assert_step_ran(run_id, "step-name").await;
test_engine.mock_step(run_id, "step-name", value).await;
```

### TypeScript Tests

- Run with `bun test` from `sdks/typescript/`
- Tests compile to `dist/*.test.js` and run with Bun's test runner

## Code Conventions

### Rust

- **Edition**: 2021
- **Style**: `cargo fmt` (rustfmt defaults)
- **Linting**: `cargo clippy --workspace`
- **Error handling**: Use `thiserror` for domain errors, `WorkflowError` for workflow failures
- **Async**: `tokio` with `async-trait` for trait methods returning futures
- **Serialization**: `serde` with `derive` feature
- **Dependencies**: Defined in workspace `Cargo.toml`, pinned versions:
  - `uuid = "=1.10.0"` (exact version)
  - `getrandom = "=0.2.15"` (exact version)

### TypeScript

- **Target**: ES2022
- **Module**: ESNext with Bundler resolution
- **Strict mode**: Enabled
- **Additional checks**: `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`
- **Schema validation**: Use `zod` for all input/output schemas
- **No runtime deps beyond zod**: Use native `fetch` and `AbortController` (Bun provides these natively)

## Workflow Implementation

### TypeScript (primary pattern)

Define workflows in TypeScript with real handlers. No Rust code needed:

```typescript
import { defineWorkflow, createWorker } from "@workflow-engine/client";
import { z } from "zod";

const myWorkflow = defineWorkflow({
  name: "my_workflow",
  input: z.object({ id: z.string() }),
  output: z.object({ result: z.string() }),
  handler: async (ctx, input) => {
    const result = await ctx.step("process", async () => {
      return `processed ${input.id}`;
    });
    return { result };
  },
});

// Auto-registers with the engine on start()
const worker = createWorker({ url: "http://localhost:8080", workflows: [myWorkflow] });
await worker.start();
```

### Rust (in-process / library embedding)

For embedding the engine directly in a Rust application without HTTP:

```rust
use workflow_engine::{Context, Workflow, WorkflowError};
use futures::future::BoxFuture;
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct MyInput { pub id: String }

#[derive(Debug, Serialize, Deserialize)]
pub struct MyOutput { pub result: String }

pub struct MyWorkflow;

impl Workflow for MyWorkflow {
    const NAME: &'static str = "my_workflow";
    type Input = MyInput;
    type Output = MyOutput;

    fn run(&self, ctx: Context, input: Self::Input)
        -> BoxFuture<'static, Result<Self::Output, WorkflowError>>
    {
        Box::pin(async move {
            let result = ctx.step("process", || async {
                Ok(format!("processed {}", input.id))
            }).await?;
            Ok(MyOutput { result })
        })
    }
}
```

## API Surface

All HTTP routes defined in `examples/http-server/src/server.rs` (router assembly) with handlers in `src/handlers/` (grouped by feature area).

### Workflow Execution

- `POST /trigger/:workflow` - Direct invocation (body: `{"input": {...}, "idempotency_key": "optional"}`)
- `POST /schedule/:workflow` - Schedule for future execution (body: `{"input": {...}, "at": "ISO8601"}`)
- `POST /publish/:event` - Publish an event (body: `{"payload": {...}, "correlation_id": "optional"}`)

### Run Management

- `GET /runs` - List runs (query: `?workflow=name&limit=N`)
- `GET /runs/:id` - Run status, output, or error
- `GET /runs/:id/history` - Step-by-step execution trace
- `POST /runs/:id/steps` - Persist a step result (body: `{"name": "...", "result": {"ok": ...} | {"err": "..."}}`)
- `POST /runs/:id/cancel` - Abort running run
- `POST /runs/:id/retry` - Retry from last checkpoint

### Admin Operations

- `POST /admin/pause/:workflow` - Reject new triggers
- `POST /admin/resume/:workflow` - Undo pause
- `POST /admin/drain` - Graceful shutdown (body: `{"timeout_ms": N}`)
- `GET /admin/dead-letters/:workflow` - List failed runs
- `POST /admin/dead-letters/:id/requeue` - Retry failed run

### Internal (Worker Communication)

- `POST /internal/workers` - Register worker and announce workflow names (body: `{"worker_id": "...", "workflows": ["name1", "name2"]}`)
- `DELETE /internal/workers/:worker_id` - Unregister worker on graceful shutdown
- `GET /internal/lease/:workflow` - Long-poll for next pending run
- `POST /internal/complete/:run_id` - Post worker result (body: `{"ok": {"output": ...}}` or `{"err": {"error": ...}}`)
- `GET /internal/events/:name` - Long-poll for a correlated event (query: `?correlation_id=<id>&timeout_ms=<ms>`; returns `200 {payload}` or `204` on timeout)

### Monitoring

- `GET /health` - Status + registered workflow names
- `GET /metrics` - Run counts by status

## Project Navigation

### Using the Engine Without HTTP

Depend on the `workflow-engine` crate directly, register Rust workflows, and call the orchestrator in-process:

```rust
let mut registry = Registry::new();
registry.register(MyWorkflow); // Rust workflow

let store = MemoryStateStore::new();
let bus = MemoryEventBus::new();
let orchestrator = Arc::new(Orchestrator::new(registry, store, bus.clone(), bus));

let run_id = orchestrator.trigger("my_workflow", input).await?;
let snapshot = orchestrator.get_run(run_id).await?;
```

See inline tests in `src/application/orchestrator.rs` for working examples.

### Building Alternative Protocol Adapters

The HTTP layer in `examples/http-server/` is one way to expose the engine. To build others:

1. Create `examples/my-adapter/` (gRPC, CLI, Kafka consumer, etc.)
2. Add to `workspace.members` in root `Cargo.toml`
3. Add `workflow-engine = { path = "../.." }` as dependency
4. Create a `Registry`, wire up adapters, start an `Orchestrator`

### Adding a New Workflow

**There is no Rust code to write.** Define the workflow in TypeScript and start a worker:

```typescript
const myWorkflow = defineWorkflow({
  name: "my_workflow",
  input: z.object({ ... }),
  output: z.object({ ... }),
  handler: async (ctx, input) => {
    // real implementation
    return { ... };
  },
});

const worker = createWorker({ url: "http://localhost:8080", workflows: [myWorkflow] });
await worker.start(); // registers my_workflow with the engine automatically
```

The engine's `/health` endpoint will show `my_workflow` in the `workflows` list immediately after `worker.start()` resolves.

### Worker Registration Protocol

Workers communicate with the engine via three internal endpoints:

| Method | Path | Purpose |
|---|---|---|
| POST | `/internal/workers` | Register worker + announce workflow names |
| DELETE | `/internal/workers/:id` | Unregister worker on graceful shutdown |
| GET | `/internal/lease/:workflow` | Long-poll for next pending run |
| POST | `/internal/complete/:run_id` | Post execution result back |

Any language can implement this protocol to execute workflows — not just TypeScript.

### Adding a New TypeScript Workflow

Define in TypeScript, pass to `createWorker`. No Rust changes needed:

```typescript
const worker = createWorker({
  url: "http://localhost:8080",
  workflows: [workflowA, workflowB, workflowC],
});
await worker.start(); // all three auto-register with the engine
```

To remove a workflow: remove it from the `workflows` array and redeploy the worker. The engine unregisters it when the worker disconnects.

### Modifying Domain Types

**Critical**: Core library modules should maintain zero HTTP dependencies. Changes to core types must not introduce:
- Network I/O (`reqwest`, `hyper`)
- File system I/O (beyond `std::fs` in tests)
- HTTP-specific types (keep HTTP concerns in `examples/http-server/`)

Only depend on trait abstractions defined in `src/ports/traits.rs`.

**Layer constraints**:
- **domain/** - No dependencies on ports, adapters, or application
- **ports/** - May depend on domain types only
- **adapters/** - Implement port traits, may depend on domain and ports
- **application/** - May depend on all layers

### Adding a New Adapter

**Storage or Event Bus Adapter:**

1. Add module under `src/adapters/` (e.g., `src/adapters/postgres/`)
2. Implement required traits from `src/ports/traits.rs` (`StateStore`, `EventPublisher`, `EventSubscriber`, etc.)
3. Export from `src/adapters/mod.rs`
4. Re-export from `src/lib.rs` if you want it available as part of the public API
5. Wire up in your composition root (e.g., `examples/http-server/src/main.rs` for HTTP, or your own entry point)

**Alternative Protocol Adapter (replacing HTTP):**

1. Create new example: `examples/my-adapter/` (e.g., `grpc-server`, `cli`, `kafka-consumer`)
2. Add to workspace members in root `Cargo.toml`
3. Depend on `workflow-engine` library
4. Implement your protocol layer (gRPC, CLI commands, message queue consumer, etc.)
5. `examples/http-server` is just one example - you don't need to modify it
6. The HTTP server follows Clean Architecture (bootstrap/composition/dto/handlers/server) — you can follow the same pattern or design differently

## Common Gotchas

### 1. TypeScript Worker Step Checkpointing

**Status**: ✅ Implemented — Option C (pre-fetch on lease + write-through cache).

**How it works**:
- On lease: `GET /runs/:id/history` pre-fetches all completed step results into a local Map
- `ctx.step()` checks the Map first (zero network on replay), then executes and writes to `POST /runs/:id/steps`
- Write-through: local cache updated immediately after each write
- Only `Ok` results are cached — failed steps re-execute on retry

**Remaining limitation**: `ctx.sleep()` is still in-process only. A worker restart mid-sleep will re-execute the sleep from the beginning rather than resuming from where it was.

### 2. Memory Adapter Durability

**Issue**: `MemoryStateStore` loses all state on server restart.

**Impact**: All runs, checkpoints, and history are lost when the server stops.

**Workaround**: Design steps to be idempotent; use client-side retry for critical flows.

**Status**: `migrations/` reserved for a future Postgres adapter (not implemented yet).

### 3. Remote Workflow Cancellation

**Issue**: `ctx.parallel` with `FailFast` cancels Rust futures correctly, but doesn't signal TS workers to stop mid-execution.

**Impact**: Cancelled remote workflow may continue running on worker side.

**Status**: Known limitation, worker cancellation signal not implemented yet.

### 4. Exact UUID Version Pinning

**Context**: `uuid = "=1.10.0"` is pinned to exact version in workspace dependencies.

**Reason**: Avoid breaking changes in UUID generation between patch versions.

**Action**: Do not change this pin without testing all workflow ID generation.

### 5. ctx.waitForEvent Ordering Constraint

**Behaviour**: `ctx.waitForEvent` long-polls the server for an event matching `correlation_id`. It only receives events published **after** the call is made — past events are not replayed.

**Pattern**: set up the wait before triggering the external action that produces the event:

```typescript
// ✅ Correct — wait first, then trigger the action
const paid = await ctx.waitForEvent(paymentReceived, { correlationId: invoiceId });

// ❌ Wrong — event may arrive before wait is established
await sendPaymentRequest(invoiceId);
const paid = await ctx.waitForEvent(paymentReceived, { correlationId: invoiceId });
```

**Default correlation ID**: `run_id` is used when no `correlationId` is specified.

## Dependencies

### Rust Workspace Dependencies

All versions managed in root `Cargo.toml` `[workspace.dependencies]`:

- `tokio = "1"` (full features)
- `axum = "0.7"`
- `serde = "1"` (derive)
- `serde_json = "1"`
- `async-trait = "0.1"`
- `thiserror = "1"`
- `uuid = "=1.10.0"` (exact, v4 + serde)
- `chrono = "0.4"` (serde)
- `futures = "0.3"`
- `tracing = "0.1"`
- `tracing-subscriber = "0.3"`
- `getrandom = "=0.2.15"` (exact)

### TypeScript Dependencies

In `sdks/typescript/package.json`:

- **Runtime**: `zod ^4.4.3` (only runtime dependency)
- **Dev**: `typescript ^7.0.2`, `@types/node ^26.1.2`

## Security Considerations

**For HTTP deployments** (using `examples/http-server`):

- **No authentication**: API and worker registration endpoints are unauthenticated. Add auth middleware in `examples/http-server/src/server.rs` before deploying publicly.
- **No authorization**: Any client can trigger any workflow. Any process can register as a worker.
- **No input validation beyond type safety**: Workflows receive any JSON matching the input schema. Add business-level validation in handlers.
- **Long-poll lease duration**: Default 30s. Workers must complete within this window or the run times out.

**For in-process embedded usage:**

Security is application-specific - the engine itself just orchestrates workflows. Apply your application's existing security model to workflow triggers.

## Performance Notes

- **Concurrency limits**: Set via `WorkflowConfig` per workflow
- **Retry backoff**: Configurable per workflow (fixed or exponential)
- **Step checkpointing**: TS `ctx.step()` persists to server; worker restarts resume from last checkpoint
- **Event fan-out**: All subscribed workflows triggered immediately on event publish
- **Memory usage**: `MemoryStateStore` keeps all run history in RAM until process restart

## Debugging Tips

### Enable Trace Logging

```bash
RUST_LOG=debug cargo run -p http-server
# or
RUST_LOG=trace cargo run -p http-server
```

### Inspect Run History

```bash
curl http://localhost:8080/runs/:id/history
```

Returns step-by-step execution trace with timestamps and results.

### Check Dead Letters

```bash
curl http://localhost:8080/admin/dead-letters/:workflow
```

Lists all failed runs for a workflow.

### Test with Virtual Clock

```rust
let (orchestrator, test_engine) = test_client(registry);
test_engine.advance_time(Duration::from_hours(1)); // instant
```

Proves time-dependent behavior (sleeps, timeouts) without waiting.

## Release Checklist

Before merging:

1. `cargo test --workspace` passes
2. `cargo clippy --workspace` has no warnings
3. `cargo fmt --all --check` passes
4. `cd sdks/typescript && bun test` passes
5. Integration test with running server + TS worker succeeds
6. No new dependencies added without justification
7. Public API changes documented in relevant files

## Future Work (Not Implemented Yet)

- **Postgres adapter**: Persistent `StateStore`, migrations in `migrations/`
- **ctx.sleep() persistence**: TS worker sleep is in-process only; restart re-executes from the beginning of the sleep
- **Remote workflow cancellation signal**: Notify TS workers of cancelled runs
- **Authentication/authorization**: Secure API and worker registration endpoints
- **Observability**: Metrics, distributed tracing
- **Horizontal scaling**: Multi-instance orchestrator with distributed locking

---
> Source: [sabryio/workflow-engine](https://github.com/sabryio/workflow-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
