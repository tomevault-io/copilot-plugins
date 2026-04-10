## concerto

> Concerto is roughly 80% complete. Treat it as a finishing and hardening phase project, not a greenfield project.

# AGENTS.md

## Project Stage

Concerto is roughly 80% complete. Treat it as a finishing and hardening phase project, not a greenfield project.

Priorities:

1. Preserve the current architecture and trust model.
2. Finish missing behavior in the existing modules.
3. Tighten validation, retry behavior, observability, and tests.
4. Avoid broad refactors unless they remove concrete complexity or unblock a required feature.

## What This Project Is

Concerto is a long-running local orchestrator that:

- polls Linear for active issues
- prepares one isolated workspace per issue
- runs Codex app-server in that workspace
- retries and reconciles runs until the issue leaves active states

Current core modules:

- `src/orchestrator.rs`: polling, dispatch, retry, reconciliation, workflow reload
- `src/agent.rs`: Codex app-server process management and turn loop
- `src/workspace.rs`: workspace lifecycle and hooks
- `src/tracker.rs`: Linear integration and issue normalization
- `src/workflow.rs`: `WORKFLOW.md` parsing, rendering, watching
- `src/config.rs`: typed config and validation
- `src/domain.rs`: shared runtime models and snapshots

Read `README.md` first. Use `SPEC.md` only when behavior is ambiguous or a larger design choice is involved.

## Operating Rules

- Keep changes simple and local.
- Prefer explicit state transitions over clever abstractions.
- Do not introduce new heavy dependencies for small features.
- Do not redesign public config shape unless the existing shape is clearly blocking required behavior.
- Preserve the high-trust v1 model unless the task explicitly asks to change it.
- Prefer deterministic behavior over “smart” implicit behavior.

## Change Guidance

When editing this repo:

- Extend existing modules before adding new ones.
- Reuse existing domain types and error variants where reasonable.
- Keep logging structured and operator-useful.
- Make failure semantics explicit. If a path is retryable or terminal, encode that clearly.
- Be careful with long-running async loops, cancellation, process shutdown, and watcher behavior.
- Keep workspace path handling strict. Avoid any change that weakens path validation or issue isolation.

Avoid:

- speculative framework layers
- generic plugin systems
- hidden background tasks without ownership/cleanup clarity
- broad renames or formatting-only churn

## Tests And Validation

Minimum validation for meaningful code changes:

```bash
cargo test
cargo clippy --all-targets --all-features -- -D warnings
```

Notes:

- The repo hook enforces the `clippy` command above.
- Integration coverage lives in `tests/integration.rs`.
- Keep tests focused on behavior, not implementation detail.
- For orchestrator/agent changes, prefer adding or extending integration-style tests over only unit tests.

## Implementation Bias

Prefer work in these areas:

- missing edge-case handling
- retry and cancellation correctness
- workflow reload safety
- tracker pagination/batching correctness
- workspace cleanup and hook reliability
- clearer logs and runtime snapshots

Be conservative in these areas:

- concurrency model changes
- config schema churn
- Codex protocol shape changes
- trust/sandbox policy changes

## If You Are Unsure

Choose the option that:

- keeps the current architecture intact
- reduces operational risk
- is easy to test with the current harness
- is easy for the next engineer to debug from logs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BubblePtr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BubblePtr)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
