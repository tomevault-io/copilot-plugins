## skelegent

> Entrypoint for any coding agent working in this repo.

# AGENTS.md

Entrypoint for any coding agent working in this repo.

`CLAUDE.md` is a symlink to this file. Both point to the same content.

## What This Project Is

Skelegent is a Rust workspace implementing a 6-layer composable agentic AI
runtime. Layer 0 defines the stability contract (protocol traits, wire types).
Layers 1–5 build implementations on top. Every concern — from provider
serialization to secret management — lives in exactly one crate.

Core values (in priority order): composability over convenience, declaration
separated from execution, slim defaults with opt-in complexity. See
`ARCHITECTURE.md` for full rationale.

## Key Abstractions

You must understand these types to work in this codebase:

| Type                                      | Crate              | Role                                                                                                                                                                                                                                          |
| ----------------------------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Operator`                                | layer0             | Object-safe trait. `execute(input, ctx) -> Result<OperatorOutput, ProtocolError>`. The unit of agent behavior.                                                                                                                                |
| `DispatchContext`                         | layer0             | Execution metadata threaded through every boundary: dispatch ID, trace context, auth, typed extensions. Every operator, tool, and middleware receives this.                                                                                   |
| `Context`                                 | skg-context-engine | Mutable conversation substrate: messages, extensions, metrics, intents. Direct synchronous mutations. Intents declared via `push_intent()` and drained into `OperatorOutput::intents`.                                                        |
| `Middleware` / `Pipeline`                  | skg-context-engine | `Middleware` is the single abstraction for context transformation. `Pipeline` holds ordered before_send/after_send middleware phases. Budget guards, compaction, telemetry are all middleware.                                                  |
| `Intent`                                  | layer0             | Executable declarations (Delegate, Handoff, Signal, WriteMemory, etc.). Operators declare intent; outer layers execute.                                                                                                                        |
| `ExecutionEvent`                          | layer0             | Semantic observation envelope: status changes, tool calls, intent declarations, artifacts, completion. Stream-first.                                                                                                                           |
| `CapabilitySource` / `CapabilityDescriptor` | layer0           | Read-only discovery. Sibling to `Dispatcher`. Describes what operators are available and what they accept.                                                                                                                                    |
| `Outcome`                                 | layer0             | Typed invocation result: Terminal, Suspended, Transferred, Limited, Intercepted.                                                                                                                                                              |
| `ProtocolError`                           | layer0             | Canonical serializable failure at invocation boundaries.                                                                                                                                                                                      |
| `Provider`                                | skg-turn           | NOT object-safe. Generic `<P: Provider>` everywhere, erased at the `Operator` boundary. Wraps LLM inference (Anthropic, OpenAI, Ollama, etc.).                                                                                               |
| `Dispatcher`                              | layer0             | Invokes operators by ID. The orchestration boundary.                                                                                                                                                                                          |

### How they connect

```
User message
  → Dispatcher.dispatch(ctx: &DispatchContext, input)
    → Operator.execute(input, &DispatchContext)
      → react_loop(Context, Provider, Tools, &DispatchContext, config, &Pipeline)
        → Pipeline.run_before(ctx) runs middleware
        → Context.compile() + Provider.infer(request)
        → Pipeline.run_after(ctx) runs middleware
        → Provider.infer(request) → response (projected into ExecutionEvents)
        → Tools execute with DispatchContext
        → Context.push_intent(Intent) declares executable intent
      → OperatorOutput { content, outcome: Outcome, intents, metadata }
    → Outer layer executes declared intents
```

## Where to Make Changes

| Task                                                     | Where                                                                   |
| -------------------------------------------------------- | ----------------------------------------------------------------------- |
| New protocol trait or wire type                          | `layer0/`                                                               |
| Change operator behavior (runtime loop, middleware, pipeline) | `op/skg-context-engine/`                                                |
| New simple operator                                      | `op/skg-op-single-shot/` or new `op/` crate                             |
| New LLM provider                                         | new `provider/skg-provider-*` crate implementing `Provider`             |
| New intent variant                                       | `layer0/src/intent.rs` (enum) + `effects/skg-effects-local/` (handler)  |
| New middleware                                           | `skg-context-engine` for context/runtime middleware; `layer0` for dispatch/store/exec middleware traits |
| New state backend                                        | new `state/` crate implementing `StateStore`                            |
| New environment                                          | new `env/` crate implementing `Environment`                             |
| Tool infrastructure                                      | `turn/skg-tool/` (trait, registry) or `turn/skg-mcp/` (MCP bridge)      |
| Orchestration patterns                                   | `orch/skg-orch-kit/` (utilities) or `orch/skg-orch-local/` (local impl) |
| Auth/secrets                                             | `auth/`, `secret/`                                                      |
| The umbrella crate                                       | `skelegent/`                                                            |

## Where Truth Lives

| What                       | Where                                                  |
| -------------------------- | ------------------------------------------------------ |
| Architectural positions    | `ARCHITECTURE.md`                                      |
| Behavioral requirements    | `specs/v2/` (current)                                  |
| Operational constraints    | `rules/`                                               |
| Deep rationale             | `specs/v2/` (see spec docs for rationale)              |

Authority: ARCHITECTURE.md > specs/v2 > rules > agent judgment. If specs are
ambiguous, update the specs (do not invent behavior).

## Load Order

Before implementation work, load in order:

1. This file
2. `ARCHITECTURE.md`
3. `SPECS.md` then the specific spec(s) for your task under `specs/v2/`
4. The relevant `rules/`

## Verification

This repo uses Nix-provided Rust tooling. All must pass before any commit:

```bash
nix develop -c nix fmt
nix develop -c cargo test --workspace --all-targets
nix develop -c cargo clippy --workspace --all-targets -- -D warnings
```

Use the Nix commands directly; there is no wrapper verification script.

For layer0 test-utils:
`nix develop -c cargo test --features test-utils -p layer0`

Do not claim "done" without fresh evidence from the relevant commands.

## Communication Hygiene

- Optimize outputs for terminal readability. Avoid excess vertical sections and
  vertical whitespace where unnecessary.

## Patterns to Know

**Intents are on Context, not a separate parameter.** Operators declare intents
via `ctx.push_intent(intent)` during execution. These are drained into
`OperatorOutput::intents` by the runtime output helpers. There is no separate
intent emitter parameter on `Operator::execute`.

**AgentOperator is the thin adapter.** It wraps `react_loop()` as a
`dyn Operator`. It forwards the `DispatchContext` it receives — it does not
fabricate one. Its config is `ReactLoopConfig`.

**Context mutations are direct.** `Context` is a mutable substrate with
synchronous mutation methods. Middleware runs through `Pipeline::run_before()`
and `Pipeline::run_after()` around inference boundaries. Budget guards,
compaction, and telemetry are middleware.

**ExecutionEvent is the semantic observation plane.** Provider chunks are projected into semantic events at meaningful boundaries — status changes, tool calls, intent declarations, artifacts, completion. Observers subscribe to these events rather than raw provider chunks.

**Provider is generic, Operator is object-safe.** `react_loop<P: Provider>` is
generic over the provider. The object-safe boundary is `Operator`, which erases
the provider type. This is by design — see ARCHITECTURE.md §"The Object-Safety
Decision."

## Codifying Learnings

When a failure mode repeats:

1. Fix the immediate issue.
2. Encode: behavior requirement → spec in `specs/v2/`. Process constraint → rule in
   `rules/`.

## Rules Index

Rules in `rules/` are numbered by concern area. Gaps in numbering are
intentional — numbers reserve space for future rules in their domain. Currently
defined: `01` (scope), `02` (verification), `04` (TDD), `06` (worktrees), `07`
(commits), `08` (review), `11` (protocol philosophy).

---
> Source: [secbear/skelegent](https://github.com/secbear/skelegent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
