## localmoe

> `docs/RUST_REWRITE_AUDIT.md` is the architecture and migration baseline for

# localMoE Rust/CUDA Rewrite Constraints

`docs/RUST_REWRITE_AUDIT.md` is the architecture and migration baseline for
this repository. Any change that alters the target architecture, migration
boundaries, or acceptance criteria must update that document in the same
change.

The V4-Pro storage/compute separation contract is
[`docs/DISAGGREGATED_EXPERT_ARCHITECTURE.md`](docs/DISAGGREGATED_EXPERT_ARCHITECTURE.md).

## Frozen README titles

- `README.md` uses `Want to Try Deploying a Full 1.6T MoE Model on Your Gaming Laptop?`;
  `README.zh-CN.md` uses `想不想在你的游戏本上试试亲自全量部署1.6T的MoE大模型呢？`.
  The visible descriptions are `Does Your Next LLM Infrastructure Really Need to Be an H100?`
  and `你的下一台LLM基础设施，何必是H100`, respectively. Agents must not change either
  title or description unless the user explicitly specifies new text.

## Rewrite target

- The end state rewrites host logic and every Colibri-owned CUDA kernel in Rust
  using `cuda-oxide`. The existing C/CUDA implementation remains only as a
  temporary correctness oracle and performance baseline during migration.
- Keep the CUDA C baseline until its corresponding Rust implementation passes
  correctness parity and the defined performance acceptance gate. Do not remove
  or regress the baseline pre-emptively.
- Support multiple open-weight model families. Model-specific variation belongs
  only in explicit abstractions such as `ModelSpec`, `Architecture`,
  `WeightMap`, `ChatTemplate`, `KvLayout`, and `MoeLayout`; runtime and kernel
  code must not hard-code a model name or weight path.

## Runtime and kernel rules

- Model checkpoints, model shards, and runtime weight caches must be downloaded
  to and read from a WSL-native ext4 block-device path. `/mnt/c`, `/mnt/d`, and
  any other DrvFS/9p-mounted Windows path are forbidden for model-weight
  storage or staging. Before a download starts, verify the target mount,
  filesystem type, usable capacity, and revision; fail closed if any check
  fails. Never silently fall back to a Windows-mounted path.
- Backend APIs batch work and use explicit streams and events. Do not add an
  implicit device/stream synchronization after a kernel launch or an
  unnecessary device-to-host round trip.
- Represent quantization formats and tensor layouts with strong types. Magic
  integer format codes are forbidden in new Rust code.
- Every performance-sensitive change requires both a correctness-parity check
  against the retained baseline and an Nsight or benchmark result.
- The initial Rust-CUDA phase targets the available Ada `sm_89` GPU. Do not
  introduce Hopper- or Blackwell-only features unless a later, separately
  validated target matrix requires them.
- Laptop-first: optimize for consumer NVIDIA laptop GPUs and low-VRAM capacity,
  with sustained (thermally stable) throughput rather than server-GPU peak
  numbers. The acceptance procedure is defined in `RUST_REWRITE_AUDIT.md`.

## Scope and delegation

- Use at most 8 subagents for a task; the platform may provide fewer concurrent slots.
- Prefer the smallest abstraction needed by the current migration phase; do
  not design speculative framework layers.
- Apply Ponytail full by default: first trace the complete path being touched,
  then follow the YAGNI ladder (existing code, standard library, native
  facilities, already-installed dependencies, then the smallest new code).
  Do not introduce a single-implementation interface, factory, one-value
  configuration, or pre-emptive scaffold.
- Keep changes to the fewest files and shortest correct diff. Do not simplify
  an explicit requirement or required safety/data-error handling. Every
  non-trivial new behavior gets one smallest runnable check. A deliberate
  simplification with a known ceiling must carry a `ponytail:` comment naming
  that ceiling and the upgrade trigger.
- For this first phase, create only the workspace/files needed for one
  end-to-end Rust-CUDA kernel. Define future model abstraction boundaries in
  the migration plan; add their code only when an implemented path uses them.
- This file intentionally does not assume a developer-specific CUDA, Rust, or
  home-directory path. Tool locations must be discovered or configured.
- The primary agent is responsible only for high-level planning, reasoning, and
  acceptance. All repository inspection, edits, builds, tests, and environment
  work are delegated to `gpt-5.6-terra` agents with high reasoning effort.

## Mechanism experiment discipline

- All experiment GPU before/after snapshots must invoke
  `rust/colibri-cuda/tools/gpu_memory_telemetry.js`; callers must not assemble
  their own `nvidia-smi` query or output columns.
- Keep the critical path explicit. A hard blocker is limited to a change in
  routes/top-k, persistent state, the throughput estimand, or data integrity.
  Record other repairs and continue without expanding the active experiment.
- Timebox debugging. Separate development evidence, decision-gate evidence,
  and claim-supporting evidence; pass a small mechanism gate before expanding
  to a full run.
- Preserve negative results and known limitations. Do not turn a
  canonical-compatible vendor reduction difference into an unbounded
  bit-matching exercise.

---
> Source: [andyzpb/localMoE](https://github.com/andyzpb/localMoE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
