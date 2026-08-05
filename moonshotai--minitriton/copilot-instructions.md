## minitriton

> English | [中文](AGENTS.zh-CN.md)

English | [中文](AGENTS.zh-CN.md)

# MiniTriton Development Rules (AGENTS.md)

This file states the engineering principles that all development on this project (human or agent)
must follow; on conflict, this file's way of working wins. Design decisions and
measured history live in the git log and the issue tracker.

## 1. Project Structure: Simplicity and Clarity

- The package body is only `minitriton/`: `frontend/ compiler/ runtime/ device/ ops/ nn/
  autograd/ distributed/ sparse/ sched/ viz/ fusion.py compile.py precision.py`. A new module
  must first answer "why does it belong in this layer"; if you can't answer, don't add a layer
- **DSL first; primitives are the exception** (decided 2026-07-19): new features / new fused ops /
  new kernels are all written in the Python DSL and lowered through the generic pipeline; the kernel
  corpus lives in the kernel library/examples, not in the compiler. Compiler intrinsics, three-way
  split: (a) tile-vocabulary primitives (the Triton-isomorphic set: load/store/dot/mma_tile/reduce/
  scan/atomic_add etc.) — frozen; (b) expressiveness exceptions (hardware instruction-level
  choreography, operations with no tile semantics, data-structure kinds) approved case by case with
  a registered justification — if you can't answer "why can't tile express this", it's not allowed;
  **explicit scheduling constructs** (async_copy/stage_pipe/pipelined/convert_layout/fragment etc.)
  belong in area (a): generic, application-independent, not application-level intrinsics; (c)
  application-level kernels (flash/KDA kinds) are forbidden in the compiler; the migration is
  **complete**: the whole flash family (fwd+bwd × all precision tiers × plain/qkv) and all three
  KDA segments are pure DSL in `device/cuda/kernels/`;
  the row-op family ships dual forms (composed reference in `sched/rowfam.py` + fused production
  kernels, routing by measurement). General linear-algebra primitives (e.g. `solve_tril`,
  a standard building block, not application-overfit) belong to the tile vocabulary; the
  forbidden kind is application-overfit intrinsics (named after specific algorithms/models) —
  current stock: **zero**
- `benchmarks/` is folded by topic: `roofline/ matmul/ attention/ kda/ ops/ training/ probes/`;
  no new files flat at the root; one-off probes go in `probes/` with filenames starting with `_`
- `examples/` holds end-to-end demos only; `build/` is compiler intermediates (gitignored, do not
  commit)
- Large figures and data files are **locally regenerable and do not enter git** (`.gitignore`
  allowlist). Git keeps only 3 headline figures (`roofline/roofline_cuda_core.{png,svg}`,
  `roofline/roofline_tensor_core.{png,svg}`,
  `training/train_gpt_convergence_precision.{png,svg}`) plus 2 gallery figures for the
  README (`examples/gallery_rigid.{png,mp4}` via `examples/render_gallery.py`,
  `examples/train_gpt_ddp_overlay.png` via `benchmarks/training/plot_ddp_overlay.py`) —
  every tracked figure must have its regeneration command on record
- imports are always `import minitriton as mt`; intra-package relative imports preferred; no reverse
  cross-layer imports (device does not import nn, runtime does not import ops)

## 2. Testing Rules

- **Every feature must ship with a PyTorch (or numpy/fp64) diff-test**: a new op/kernel delivery =
  implementation + a reference test under `tests/` (torch is only allowed in `benchmarks/`;
  `tests/` always uses numpy/fp64 as the oracle; acceptance scripts needing a torch reference go in
  `benchmarks/`)
- State tolerances explicitly with justification (fp32 kernel vs numpy: rtol ~1e-5; tf32: 1e-2;
  bf16: 2e-2; EXC ill-conditioned regions only assert finiteness)
- The test matrix must cover: regular shapes, odd/non-divisible, non-contiguous views, broadcasting,
  boundaries (0-dim, single element), dtype variants
- Delivery bar: `pytest tests/ -q` all green + new cases pass; performance kernels must additionally
  meet "compute-bound ≥ 0.9x triton like-for-like"
- Fast parallel gate (pytest-xdist + pytest-rerunfailures are in the project env; the compile disk
  cache is concurrency-safe by design):
  `pytest tests/ -q -n 12 --deselect tests/test_streams.py --reruns 2 --reruns-delay 3`
  then `pytest tests/test_streams.py -q` serially (the streams tests are timing-sensitive under
  GPU contention). ~20 s warm vs ~75 min serial cold-after-compiler-edits
- autograd-related: `benchmarks/training/grad_check.py` (vs torch autograd, atol 1e-4) must not
  regress

## 3. Plotting Rules

- All figures go through the unified system in `benchmarks/roofline/plot_style.py`:
  - **Six-version output**: `ps.render_themes(fig_fn, out_base)` produces (dark, light) × (png,
    svg, pdf); `fig_fn` uses only `ps.*` constants and is re-callable. Light PDFs are the
    print/LaTeX-appropriate variants, dark PDFs suit slides
  - **Single color vocabulary**: `IMPL_COLORS` (minitriton hero red / torch blue / torch.compile
    green / triton purple); no new implementation colors
  - **Split legends**: `ps.split_legend` — method (color) and kernel (marker) as two independent
    legends, never mixed
  - **roofline must draw the measured roof** (self-calibrated from measurements, not spec-sheet
    numbers) **and the spec dashed line** (boost values, to preempt nitpicking); guidance
    annotations like "closer to the roofline = better" must be restrained
- **Data first**: every point on a figure must come from a csv/json intermediate (scripts write data
  first, then plot), and provide a zero-GPU replot path (`REPLOT=1`); figures and data must never
  drift apart
- **All training curves must overlay the PyTorch reference**: same init / same batch / same lr
  schedule, both curves on the same figure; the convergence alignment acceptance standard = last-50
  mean gap < 0.01 (methodology and measured envelopes: the scripts under `benchmarks/training/`)
- Put legend text directly on the series where possible (callouts), reducing corner legends; no
  small-print footnotes at the bottom

## 4. Code Delivery Rules

- **Core sizing (set 2026-07-16)**: small shapes (≤1024³ / T≤512) and e2e (mixed / full-bf16) are
  not one-shot push targets; stair acceptance (P1 1.0→1.3x, P2 0.9→1.1x, P3 ≥1.2x; may exceed, may
  not regress); before the next stair starts, the current one must be explainable / reproducible /
  attributable (PTX/SASS + roofline + per-op breakdown, no blind tuning); red gates = no silent
  mainline regressions, pytest default suite all green; numbers must be backed by CSV + figures +
  references


- Branch model: `main` for continuous integration, `stable` as the stable reproduction baseline
  (advances after main passes acceptance); feature branches `phaseX-*` / `topic-*`, merged by the
  main session after acceptance
- commits in English, single purpose; WIP may be committed but must keep `pytest` all green
- Delivery = code + tests + benchmark data + doc sync (README and this file stay in sync with
  the code); missing any piece means not done
- Forbidden: tests/ importing torch; runtime code depending on triton; "merge first, test later"
  without tests; touching stable before the main session merges; adding application-level compiler
  intrinsics (§1 three-way split: application kernels are always written in DSL)
- The default fast path must never regress for precision/experimental modes; precision capabilities
  are always opt-in env switches (paradigm: see `minitriton/precision.py`)

## 5. Communication Rules with Developers

- Report only **actually measured** results, and paste the verbatim output of key commands (pytest
  tail, benchmark tables, git log); mark unverified things as "unverified"
- Performance conclusions must come with a reference table (tl / torch eager / torch.compile /
  triton like-for-like) and the test environment (GPU model, card index, precision tier, timing
  method)
- When blocked, state facts first (what error, which step), without speculative attribution; mark
  bugs that can't be reproduced as "not reproduced"
- Announce before changing files the user is currently viewing; all intermediate states can be
  committed at any time; don't hide the working tree

---
> Source: [MoonshotAI/minitriton](https://github.com/MoonshotAI/minitriton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
