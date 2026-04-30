## rustane

> **Read `dev/CURRENT.md` first** — it has the live project state, recent fixes, active experiments, and what to do next.

# CLAUDE.md

## Start Here
**Read `dev/CURRENT.md` first** — it has the live project state, recent fixes, active experiments, and what to do next.

## What This Is
Rust training + inference engine for Apple Neural Engine (ANE) + Metal GPU. 48.8M param GPT (gpt_karpathy) pretraining on M4 Max. Memory-safe, portable to Jetson via candle-rs + CUDA.

## Crates
- **`ane-bridge`** — Rust FFI to ANE private APIs via dlopen
- **`metal-decode`** — Metal shaders for single-token decode
- **`engine`** — Training orchestrator: ANE forward, CPU backward, Metal Adam

## Commands
```bash
cargo build                                              # build
cargo test -p engine --release                           # all engine tests
cargo test -p engine --test phase4_training --release -- --nocapture  # training validation
cargo test -p engine --test ab_fix_isolation --release -- --ignored --nocapture  # A/B tests
cargo run -p engine --release                            # training binary
```

## Key Files
- `crates/engine/src/full_model.rs` — forward/backward/train_step, TrainConfig defaults
- `crates/engine/src/layer.rs` — per-layer forward/backward with ANE kernels
- `crates/engine/src/cpu/vdsp.rs` — Accelerate FFI (sgemm_at uses beta=1.0, accumulates)
- `crates/engine/src/bin/train.rs` — training binary with CLI flags

## Tracking
- **Current state**: `dev/CURRENT.md` (read first every conversation)
- **Experiments**: `system/experiments.tsv` (structured: date, name, variable, result, verdict)
- **Results index**: `results/INDEX.md` (one-line summaries)
- **Methodology**: `dev/METHODOLOGY.md`
- **Automation**: `system/optimize-loop.sh` (autonomous optimization agent)
- **Credits**: `CREDITS.md`

## ANE Gotchas
- IOSurface spatial width must be multiple of 16 (silent data corruption otherwise)
- ANE compiler fails on rsqrt/sqrt after reduce ops — use pow(-0.5)
- sgemm_at uses beta=1.0 — always zero output buffer before calling
- Per-ANE-dispatch overhead: ~0.095ms (XPC + IOKit round-trip)

## Metrics
- **ms/step** — training step time (target: match maderix 89-120ms)
- **val_bpb** — validation bits per byte (quality metric)
- **tok/s** — decode throughput

---
> Source: [ncdrone/rustane](https://github.com/ncdrone/rustane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
