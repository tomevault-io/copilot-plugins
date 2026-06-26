## clawmark

> This file is the entry point for AI agents working on this codebase.

# clawmark — Agent Context

This file is the entry point for AI agents working on this codebase.

## Communication

Be concise. Answer the question directly; skip preamble, long tables, and repeated summaries unless the user asks for depth.

## Project

`clawmark` is a local Rust CLI that A/B tests exactly two `CLAUDE.md` files against SWE-bench Lite. See [CLAUDE.md](./CLAUDE.md) for project-level coding guidelines.

## Implementation Plan

The full phased implementation plan (architecture, types, security rules, milestones) lives in a local `PLAN.md`.

`PLAN.md` is gitignored — it is a maintainer-local working document and is not committed, so it will not be present in a fresh public clone. The committed `README.md` and the per-module doc comments are the source of truth for contributors.

## Priorities

**Minimize dependencies.** Before adding any crate, ask: can this be done with std? If a dep saves fewer than ~50 lines of non-trivial code, use std instead. Every new dependency must be justified in PLAN.md's dependency rationale table before it is added to `Cargo.toml`.

**No scope creep.** Implement exactly what PLAN.md specifies for the current phase — nothing more. Do not add flags, config fields, abstractions, or error variants that aren't required by the phase's deliverables. If something seems like a natural extension, note it in a comment or TODO, but do not implement it.

**Protect the minimal v1.** v1 is only `doctor`, `run --a <path> --b <path> --model <model> [--model-a <model>] [--model-b <model>] --timeout-secs <seconds> --out <dir> [--parallel N]`, and `report --out <dir> [--show-patches]` for local SWE-bench Lite A/B testing. Do not add N-variant matrices, repeated trials, pass@k, McNemar/Wilson statistics, resume/retry systems, plugin APIs, cloud/remote execution, progress UI, release automation, or aggregate cost caps unless PLAN.md is intentionally expanded first. Per-variant time, token, and cost REPORTING is in scope (intentionally expanded); budget enforcement, per-task cost caps, and whole-run accounting beyond reporting remain out of scope.

**Prefer direct A/B concepts.** Use fixed variant slots `A` and `B`, simple output files (`a.jsonl`, `b.jsonl`, `a.json`, `b.json`), and win/loss/tie report metrics. Avoid generic benchmark abstractions until there is a second benchmark target.

**Ship the smoke runner first.** If only one thing can work, it must be the reliable bundled 5-task A/B smoke runner. Do not add partial support for all 300 SWE-bench Lite tasks in v1.

**Use the fixed smoke set.** v1 runs exactly `astropy__astropy-12907`, `astropy__astropy-14182`, `astropy__astropy-14365`, `astropy__astropy-14995`, and `astropy__astropy-6938`, in that order. `data/swebench_lite_v1_subset.jsonl` must contain exactly those five unique IDs.

**CLI-only v1.** Require run parameters on the CLI. Do not add `clawmark.toml`, config-file parsing, or the `toml` crate.

**Prefer fewer dependencies over richer internals.** Use plain error messages and empty patches for unresolved outputs. Do not add `anyhow`, `thiserror`, or model-output diff extraction in v1. `tokio` is used for concurrent Claude invocations when `--parallel > 1`; harness invocations remain serial (`std::process::Command`).

**Keep output semantics simple.** `run --out <dir>` must fail if `<dir>` already exists; do not append, clear, resume, or partially reuse previous output. `report --out <dir>` only reads existing v1 output.

**Parse harness results narrowly.** For v1, `report` reads `<out>/harness/a.json` and `<out>/harness/b.json` and treats their `resolved_ids` arrays as the only source of truth. Do not parse alternate SWE-bench output schemas unless PLAN.md changes.

## Quick orientation

- Language: Rust (edition 2021)
- Benchmark: SWE-bench Lite only (v1)
- Task set: bundled 5-task smoke subset only, fixed to the IDs listed in PLAN.md
- Variants: exactly two local files, addressed as A and B
- Execution: up to `--parallel N` Claude invocations concurrently within each variant pass (default N=1, sequential); harness always runs serially, one invocation per variant
- Reporting: resolved counts plus A wins, B wins, ties, and both-failed counts; per-variant wall-clock time (seconds), input tokens, output tokens, and estimated USD cost (shown as `n/a` when the Claude CLI did not provide it); `--show-patches` displays each task's patch truncated to 20 lines; failure summary printed automatically when any task is unresolved
- Claude invocation: `claude -p --output-format json --dangerously-skip-permissions --model <model> --add-dir <workspace> -- <problem_statement>` (note: `--bare` is intentionally NOT used; see the comment in `src/runner.rs`). A and B may use different models via `--model-a`/`--model-b`; both default to `--model` when the per-variant override is absent.
- Docker: required (SWE-bench harness uses per-instance containers)
- No shell strings in subprocesses — always `Command` argv arrays (harness) or tokio `Command` (Claude, when parallel)
- Empty `git diff HEAD` means unresolved; write an empty patch
- Failure records need only `error: Option<String>`, not typed categories
- Variant paths must canonicalize inside the current working directory and A/B must be different files
- `unsafe_code = "forbid"` enforced via `[lints.rust]`

---
> Source: [emiliolugo/clawmark](https://github.com/emiliolugo/clawmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
