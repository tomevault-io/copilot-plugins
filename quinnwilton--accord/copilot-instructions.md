## accord

> Protocol contracts for Elixir in the lineage of Joe Armstrong's UBF (2002): one specification, checked twice — a runtime blame-assigning monitor and a TLA+ model checked by TLC.

# Accord

Protocol contracts for Elixir in the lineage of Joe Armstrong's UBF (2002): one specification, checked twice — a runtime blame-assigning monitor and a TLA+ model checked by TLC.

## What it does

Accord lets you declare a two-party protocol (states, transitions with typed replies, server-push events, guards, `where:` reply constraints, tracks, a property algebra) using a compile-time DSL. At compile time, it:

1. Builds an intermediate representation (IR) from the DSL.
2. Validates the IR through a pipeline of passes (errors E001–E061, warnings W001–W005).
3. Generates a runtime monitor (gen_statem proxy) that checks messages at runtime, with blame derived from message direction (`:client` | `:server` | `:property`, labeled with declared role names). Monitors can arm `state_timeout` transitions and capture traces (`trace_sink:`).
4. Generates a TLA+ specification for exhaustive model checking with TLC — strict by default (`tla: :strict | :warn | :off`), including multi-session checking (`sessions: N` with `shared:` tracks, `exclusive`, and `distinct`). Captured runtime traces replay against the spec (`Accord.TLA.TraceSpec`, ADR-010); the exerciser samples one per run (`trace_check:`).

`mix accord.check` caches passes by content hash (`--force` bypasses), reports stats, dumps raw TLC output (`--dump`), and checks hand-written extension modules (`--extra`). Protocol constants (`const :name, value`) substitute at compile time into every captured expression. `mix accord.compat` gates rolling upgrades: snapshot the released IR, then check old-client compatibility against every change (C001–C010, ADR-011).

## Architecture

For the two-pipeline design, blame semantics, the runtime-vs-TLC coverage table, closure lifting, pass ordering, error reporting, and module dependency rules, see [`docs/architecture.md`](docs/architecture.md). Design decisions live in [`docs/decisions/`](docs/decisions/) (ADR-001–011); the redesign rationale and soundness review are in [`docs/design-review-2026-07.md`](docs/design-review-2026-07.md) and [`docs/soundness-review-2026-07.md`](docs/soundness-review-2026-07.md), and open expressiveness work in [`docs/design-expressiveness-roadmap.md`](docs/design-expressiveness-roadmap.md).

The IR (`Accord.IR`) is the single source of truth. Two pipelines flow from it:

- **Downward (runtime)**: IR → transition table + track init → `Accord.Monitor` (gen_statem)
- **Upward (verification)**: IR → state space → actions → properties → TLA+ `.tla` + `.cfg`

Coverage is deliberately asymmetric and always declared: constructs TLC cannot express are labeled runtime-only (W004); ghost constructs the runtime cannot enforce (`shared:` tracks, `exclusive`) are warned about at `sessions: 1` (W003). Never silent.

### Key modules

| Module | Responsibility |
|--------|---------------|
| `Accord.Protocol` | DSL macros (`on`/`cast`/`event`/`track`/`model`/`property`), `@before_compile` orchestration, `tla:` strictness |
| `Accord.IR` | IR structs (State, Transition, Branch, Track, Property, Check, Role) + inline model config |
| `Accord.Monitor` | Runtime gen_statem proxy: calls, casts, server-push events (`:downstream`), blame assignment |
| `Accord.Pass.*` | Validation passes (structure, roles, tracks, types, determinism, reachability, properties, field paths, span refinement) |
| `Accord.Pass.TLA.*` | TLA+ passes (BuildStateSpace, BuildActions, BuildProperties, Emit) + `Expr` (fidelity errors E041–E045) |
| `Accord.TLA.Compiler` | Orchestrates TLA+ passes; returns `{:error, reports}` on infidelity |
| `Accord.TLA.Sessions` | Multi-session emission context (per-session vs shared variables) |
| `Accord.TLA.GuardCompiler` | Elixir AST → TLA+ expressions; returns emitted identifiers for E043 validation |
| `Accord.TLA.ModelConfig` | Finite domains: inline (track `domain:`, `model do`) > protocol file > project file > builtins |
| `Accord.TLA.Files` | Single source of `_build/accord/` artifact paths (write/delete) |
| `Accord.TLA.TLCParser` / `ViolationReport` | TLC stdout → structured results → pentiment diagnostics |
| `Accord.Violation` | Violation structs with direction-derived blame |
| `Accord.Compat` | Static rolling-upgrade simulation over two IRs (C001–C010) + `Snapshot` persistence |
| `Accord.Test.*` | Shipped harnesses: ProtocolExerciser (propcheck + sampled trace replay), TLACheck (real TLC assertions), TraceCheck, FaultyServer |

### Closure lifting

Guards, updates, invariants, and other user-provided functions are anonymous closures at macro expansion time. They reference the temporary `elixir_compiler_N` module, which doesn't exist at runtime. `lift_closures/2` in `Accord.Protocol` replaces each closure with a named function capture (`&Module.__accord_fn_N__/arity`) so they serialize correctly via `term_to_binary`.

## Development commands

```bash
mix test                      # run all tests (includes :tlc tests when tla2tools.jar is available)
mix test --only tlc           # only the real-TLC integration tests
mix format                    # format code
mix dialyzer                  # static analysis
mix accord.check              # model-check all protocols (deadlock on by default)
mix run examples/job_queue/demo.exs   # run an example demo
```

## Examples

`examples/` compiles in dev and test (never :prod), so protocol compilation — full validation plus strict TLA+ emission — runs on every example in CI. Each pedagogical example ships a correct server, a seeded design bug pinned by a real TLC run in `test/examples/`, and a demo script. Pedagogical order: circuit_breaker → payment → job_queue (flagship) → genstage_demand. The multi-session showcase is `test/support/protocols/distributed_lock.ex`.

Two examples run the monitor against **real third-party servers** (as dev/test deps): `http2` wires the stream monitor between a raw client and a live Bandit to catch RFC 9113 divergences (see `docs/case-study-http2-bandit.md`), and `phoenix_channels` model-checks the Phoenix Channels lifecycle and taps a real Phoenix endpoint via a one-line wrapping serializer to catch channel-protocol bugs Phoenix leaves unenforced (see `docs/case-study-phoenix-channels.md`).

## Testing conventions

- Unit tests mirror `lib/` structure in `test/accord/`; example tests in `test/examples/`.
- Property-based tests (propcheck statem) live in `test/property/`.
- Test support modules (protocols, servers) in `test/support/`; reusable harnesses ship in `lib/accord/test/`.
- TLC-dependent tests carry `@tag :tlc` + `:tmp_dir`; `test_helper.exs` excludes them when `tla2tools.jar` is absent.
- Use `@moduletag :capture_log` for tests that trigger logged violations.
- Use `:sys.get_state/1` as a synchronous barrier instead of `timer.sleep`.

## Patterns to know

- `Accord.Pass.Helpers` provides shared `maybe_add_source/2`, `maybe_add_span_label/3`, and `message_tag/1` for validation passes. Import it rather than duplicating.
- The `message_tag/1` in `Monitor` and `TransitionTable` operates on raw runtime messages (atoms/tuples) — semantically different from the IR helpers version.
- Validation passes return `{:ok, ir}` or `{:error, [Report.t()]}`. The compiler pipeline in `Protocol.compile_ir/2` chains them with `with`.
- TLA+ passes return `{:ok, result}` or `{:error, [Report.t()]}`; anything the model cannot faithfully express must be a loud, span-carrying diagnostic — never a silent drop (ADR-007).
- Branch selection (runtime) mirrors the model: first branch whose type matches AND whose `where:` constraint holds; constraint failures fall through before being reported as `:constraint_failed`.
- Multi-session emission is gated on `Sessions.multi?/1`; `sessions: 1` output must remain byte-identical (pinned by tests in `test/accord/tla/sessions_test.exs`).
- New DSL macros need `.formatter.exs` `locals_without_parens` entries.

---
> Source: [QuinnWilton/accord](https://github.com/QuinnWilton/accord) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
