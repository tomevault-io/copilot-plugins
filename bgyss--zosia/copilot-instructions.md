## zosia

> When writing complex features or significant refactors, use an ExecPlan (as described in the template in`.agent/PLANS.md`) from design to implementation. Then follow the templated plan (as specified in `.agent/execplans`) to implement the feature or refactor.

# AGENTS.md

## ExecPlans

When writing complex features or significant refactors, use an ExecPlan (as described in the template in`.agent/PLANS.md`) from design to implementation. Then follow the templated plan (as specified in `.agent/execplans`) to implement the feature or refactor.

## Project

**Name:** zosia
**Purpose:** Boot a minimal Linux VM directly into a GPT prompt, with no traditional shell/UI.

## Primary user story

As a user on macOS, I can launch a VM (QEMU initially) that boots straight into an interactive chat prompt backed by a local GGUF model.

## Goals

1. AArch64 (Apple Silicon) VM boots to console and launches an LLM REPL automatically.
2. Model stored on attached virtual disk (not embedded in initramfs).
3. Single-command run script for QEMU on macOS.
4. Reproducible builds (pinned toolchains; deterministic artifacts).

## Non-goals (v1)

- Bare-metal install/boot on Mac hardware.
- GUI/web UI.
- Tool-calling, browsing, or networking features beyond what’s needed to load/run the model.

## Sample directory structure

```text
.
├── AGENTS.md
├── README.md
├── docs/
│   ├── index.md
│   ├── quickstart.md
│   ├── architecture.md
│   └── troubleshooting.md
├── mkdocs.yml
├── scripts/
│   ├── run-qemu-aarch64.sh
│   ├── build-buildroot.sh
│   ├── build-llama.sh
│   └── make-model-disk.sh
├── buildroot/
│   ├── configs/
│   └── output/
├── initramfs/
│   └── init
├── assets/
│   └── uefi/
└── .agent/
    ├── PLANS.md
    └── execplans/
        └── 0001_mvp.md
```

## Key decisions

- Runtime: llama.cpp (llama-cli) with Phi-3.5 mini Q4_K_M as the default GGUF.
- OS: minimal Linux kernel + initramfs (busybox) generated via Buildroot.
- Guest I/O: virtio-blk for model disk; serial console for interaction.
- Docs: MkDocs for documentation in docs/.
 - Validation: gpt-oss-20b tested (MXFP4), but Phi-3.5 mini chosen for interactive speed.

## Constraints

- Must run in QEMU on macOS (Apple Silicon) with UEFI.
- Keep boot time < 5s after kernel load (excluding model mmap warmup).
- Keep the userspace minimal: only what’s required to start the REPL.
- Keep README.md and docs/ up to date with any code or behavior changes.
- When rebuilding the model disk, default to: `./scripts/make-model-disk.sh ./models/Phi-3.5-mini-instruct-Q4_K_M.gguf --use-docker`

## Performance testing

- Single perf run: `./scripts/ci/perf-boot.sh` (defaults to `PERF_MIN_TOK_S=20`)
- Phi-3.5 mini profile (tokens=128): `PERF_PROFILE=phi35_fast PERF_MIN_TOK_S=20 ./scripts/ci/perf-boot.sh` (default)
- Continuous tuning loop:
  - `./scripts/ci/gen-perf-matrix.sh` (writes `perf.matrix`)
  - `PERF_MATRIX_FILE=perf.matrix PERF_LOOP=1 PERF_FAIL_FAST=0 PERF_RESULTS=/tmp/zosia-perf.csv ./scripts/ci/perf-boot.sh`
- Tuning overrides:
  - `PERF_THREADS`, `PERF_BATCH_THREADS`, `PERF_LLAMA_ARGS` (comma-separated llama args)
  - `PERF_RUNS`, `PERF_LOG_DIR`, `PERF_TIMEOUT`, `PERF_TOKENS`

---
> Source: [bgyss/zosia](https://github.com/bgyss/zosia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
