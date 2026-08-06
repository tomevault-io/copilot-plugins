## trajeckt

> Engineering rules for contributors and AI coding agents working on `trajectoryd`.

# AGENTS.md

Engineering rules for contributors and AI coding agents working on `trajectoryd`.

`trajectoryd` is a deterministic, low-latency runtime enforcement daemon for path-dependent AI agent tool execution. It is not a generic API gateway, observability dashboard, or governance UI. Its core invariant is that enforcement decisions depend on trajectory history, not isolated requests.

## Product Invariants

- Preserve trajectory-aware evaluation. Never regress enforcement into stateless per-request checks.
- Treat the session trajectory as the enforcement subject. New checks must evaluate the accumulated path, including prior actions, delegation state, capability accumulation, boundary crossings, budgets, and corpus semantics where relevant.
- Keep enforcement deterministic. Identical policy, input trajectory, and runtime state must produce identical decisions.
- Keep the daemon focused on runtime enforcement. Do not add unrelated dashboard, generic gateway, governance, analytics, or product-surface features unless explicitly requested.
- Preserve append-only corpus semantics. Historical enforcement records must not be mutated or silently rewritten.

## Rust Standards

- Use production Rust style: clear ownership, explicit lifetimes only where useful, small cohesive modules, typed domain boundaries, and no avoidable global state.
- Do not use `unwrap()`, `expect()`, `panic!`, `todo!`, or `unimplemented!` in production paths.
- Use typed errors with `thiserror` for library and domain errors.
- Use `anyhow` only at binary or CLI boundaries, and only where it is already part of the local pattern.
- All public structs, enums, traits, functions, and modules must have rustdoc comments.
- Comments must explain why a decision exists, what invariant it protects, or what failure mode it prevents. Do not restate what the code already says.
- Prefer explicit types for domain concepts over raw strings, booleans, or loosely shaped maps when the value participates in enforcement.

## Input And State Handling

- Validate all external inputs, including configuration files, HTTP payloads, headers, policy fields, session identifiers, tool names, costs, timestamps, and upstream responses.
- Reject malformed, incomplete, ambiguous, or out-of-range inputs before they enter enforcement state.
- Bound attacker-controlled sizes. Do not allow unbounded request bodies, session histories, strings, vectors, maps, graph depth, or corpus records.
- Treat persistence as failure-aware. Writes must surface errors to callers, preserve data integrity, and avoid partial writes where possible.
- Prefer atomic write patterns, append-only records, checksums/signatures, fsync-equivalent durability, or recovery metadata where persistence correctness depends on them.
- Never silently drop enforcement state or corpus records. If state cannot be persisted or recovered, return an explicit error or fail closed where enforcement safety requires it.

## Enforcement Hot Path

- Hot-path enforcement code must avoid unnecessary allocation, blocking I/O, expensive regex, and avoidable cloning.
- Do not perform network calls, filesystem writes, heavyweight parsing, or telemetry export inside the decision path unless the design explicitly accounts for latency and failure behavior.
- Keep checks proportional to trajectory size. If a check scans history, justify the bound or maintain incremental state.
- Avoid regex for classification in the hot path unless it is precompiled, bounded, and measurably acceptable.
- Do not add locks around enforcement state without considering contention, poison behavior, latency impact, and failure handling.

## Testing

- Tests must be deterministic. Avoid wall-clock timing, random ordering, real network dependencies, and shared mutable external state unless controlled by the test.
- Tests must include failure cases, not only allowed-path examples.
- Every feature must include focused unit tests and integration tests where appropriate.
- Enforcement changes must test path-dependent behavior, including sequences where isolated actions would look safe but the accumulated trajectory must be challenged, throttled, quarantined, blocked, or terminated.
- Persistence changes must test write failures, malformed records, recovery behavior, and append-only guarantees where applicable.
- Input validation changes must test malformed, missing, boundary, and oversized inputs.

## Required Checks

Run these before marking work complete:

```sh
cargo fmt
cargo clippy --all-targets --all-features -- -D warnings
cargo test
```

If a required check cannot be run, document the exact command, the failure reason, and the remaining risk.

---
> Source: [beebeeVB/trajeckt](https://github.com/beebeeVB/trajeckt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
