## hyper-scatter

> This repo is a **correctness + performance lab** for an embedding scatterplot that supports both:

# viz-lab: Candidate Implementation Engineer Guide

This repo is a **correctness + performance lab** for an embedding scatterplot that supports both:

- **Euclidean** 2D scatter
- **Hyperbolic embeddings in the Poincaré disk** (geometry-aware camera + interactions)

This document is for working on the **optimized candidate renderer**.

If you are working on the **harness** (benchmarks, reference, runners, CI-ish glue), use **`CLAUDE.md`**. If you need to change harness semantics, flag your concerns to the user if it limits your performance potential. The user will pass on your concerns to the harness engineer.

For the full spec and rationale, see **`TASK.md`**.

---

## Where you should work (candidate code)

The candidate path already exists and is wired into the benchmark/accuracy harness:

- `src/impl_candidate/`

The reference implementations (ground truth) are:

- `src/impl_reference/euclidean_reference.ts`
- `src/impl_reference/hyperbolic_reference.ts`

Core math (shared by reference + candidate; **do not “approximate” this in the candidate**):

- `src/core/math/euclidean.ts`
- `src/core/math/poincare.ts`

---

## How the candidate is selected in the harness

The browser harness supports two renderer modes:

- `reference`: Canvas2D reference implementations
- `webgl`: WebGL2 candidate implementations

Selection happens in **`src/benchmarks/browser.ts`**:

- For performance benchmarks (`runBenchmarks`):
  - `renderer: 'webgl'` ⇒ `EuclideanWebGLCandidate` / `HyperbolicWebGLCandidate`
  - `renderer: 'reference'` ⇒ `EuclideanReference` / `HyperbolicReference`
- For accuracy tests (`runAccuracyBenchmarks`):
  - reference uses the visible `#canvas` (Canvas2D)
  - candidate uses a separate hidden/alternate `#candidateCanvas` (WebGL)

If you add a new candidate class/file, you typically only need to adjust the imports/constructor selection in `src/benchmarks/browser.ts`.

---

## Fast iteration loop (correctness first, then performance)

The harness provides:

1) **Correctness gates** (reference vs candidate accuracy)
2) **Performance benchmarks** (render + interaction workloads)

Operational expectations (read this literally — it will save you time):

- Prefer running benchmarks via the **CLI runners** (reproducible, scriptable, fewer UI variables).
- For **performance** numbers, **do not** use **headless** runs as the source of truth.
  - Headless mode can change GPU scheduling / RAF timing and can significantly skew results.
  - **Use headed (non-headless) runs** for any number you intend to compare, report, or log.
  - Headless is acceptable for quick “does it run” smoke checks or CI automation.

As a candidate engineer, your iteration loop should be:

- run the accuracy suite after any semantics-affecting change
- only optimize after you’re passing correctness
- re-run benchmarks frequently to catch regressions
- when adding a performance trick, always benchmark **before and after**, and write the results down (see the optimization log below)

---

## Running the benchmark suite (copy/paste)

All commands are run from the repo root.

### 0) Install deps (once)

- `npm install`

### 1) Correctness gate (browser accuracy)

This compares **reference (Canvas2D)** vs **candidate (WebGL)** behavior.

- **Headed (recommended when debugging):**
  - `npm run bench:accuracy`
- **Headless (CI / smoke only):**
  - `npm run bench:accuracy -- --headless`
  - Optional knobs: `--dpr=2`, `--timeout=180000`

If this fails, fix correctness before looking at performance numbers.

### 2) Performance benchmarks (browser, automated)

This launches a browser via Puppeteer, drives interactions, and prints a table.

- **Candidate (WebGL) — headed (default; required for real perf):**
  - `npm run bench`
  - or explicitly: `npm run bench -- --renderer=webgl`

- **Reference (Canvas2D) — headed:**
  - `npm run bench -- --renderer=reference`

Common overrides (apply to either renderer):

- Limit point counts (faster iteration):
  - `npm run bench -- --points=10000,100000,500000`
- Run only one geometry:
  - `npm run bench -- --geometries=euclidean`
  - `npm run bench -- --geometries=poincare`
- Increase timeout for large-N runs:
  - `npm run bench -- --timeout=240000`

Headless controls (use intentionally):

- **Force headed:** `npm run bench -- --headed` (or `--no-headless`)
- **Force headless (CI / smoke only):** `npm run bench -- --headless`

Notes:

- The runner starts its own Vite dev server on a fixed port and will open a controlled browser window.
  Avoid interacting with that window during a run.
- The printed table includes dataset generation time, CPU submit time, frame interval (FPS), pan/hover FPS, hit-test time, lasso time, and memory.

### 3) Manual UI (optional)

If you want to see the benchmark UI and run things by clicking buttons:

- `npm run bench:browser`

For deeper details on what each metric means, see `src/benchmarks/README.md`.

---

## Keep a running optimization log (recommended)

Keep a short, append-only log of what you tried and what happened. This saves time when you revisit the project or hand it off.

- Suggested location: `research/candidate_optimization_log.md`
- What to record: change summary, point counts/geometries tested, results (FPS / ms), correctness notes, and whether the technique was a win/loss.

When you introduce a performance technique/trick, the log should include:

- the **before** benchmark numbers (same scenario/config)
- the **after** benchmark numbers
- any observable trade-offs (quality, memory, cross-browser behavior)

---

## Hyperparameters: tune by measurement, not intuition

The candidate will likely have “knobs” (hyperparameters) such as thresholds, budgets, DPR floors, LOD cutovers, index granularities, etc.

Guidelines:

- Avoid setting hyperparameters purely based on intuition.
- Prefer running **benchmark sweeps** across reasonable ranges to find robust values.
- Don’t overfit hyperparameters to a single machine/browser/GPU; choose values that generalize to major browsers and good consumer hardware.
- Validate knobs on **both** geometries and multiple point counts (small + large), and record the sweep results (or at least the final rationale) in `research/candidate_optimization_log.md`.

---

## Product constraints to keep in mind

Even though the solution is validated in this harness, it will ship in a **web-based product**:

- It should work across major browsers and “good consumer hardware” (not just one dev GPU/driver).
- Avoid relying on quirks that only work in one browser/GPU stack.
- The harness runs on a representative machine to approximate real conditions, but you should still keep portability in mind.

Also: keep aesthetics in mind (pleasant point rendering, sensible colors, avoid obvious artifacts), but **prioritize performance** and interaction smoothness.

---

## What must remain exact (candidate invariants)

The candidate may change **how things are rendered**, but must preserve **what the system means**.

### Required correctness properties

- **Project/unproject** must match reference within tolerance.
- **Pan/zoom** must be **anchor-invariant** (cursor anchor stays fixed under the drag/zoom semantics defined by the geometry).
- **Hit-test** must match exactly (including deterministic tie-breaks).
- **Lasso selection** must match exactly.

Default tolerances enforced by the harness are documented in `src/benchmarks/README.md`.

### Important nuance: render quality vs interaction semantics

The existing WebGL candidate uses adaptive quality (LOD / lower offscreen DPR / squares vs circles) to keep interaction smooth at large $N$.

This is OK **only if it affects rendering output**, not semantics:

- hit-testing and lasso selection must remain exact (CPU-side) and match reference
- project/unproject + view state updates must remain exact (delegated to `src/core/math/*`)

---

## “If you touch X, check Y” (practical checklist)

- If you change camera/view math: run the **accuracy suite** immediately.
- If you change hit-testing or lasso: run the **accuracy suite** and spot-check edge cases (ties, near-boundary points).
- If you change GPU buffer upload strategy (subsampling, caps like `maxGpuUploadPoints`):
  - confirm semantics still use the full dataset on CPU
  - confirm selection/hover overlays don’t OOM for huge selections
- If you modify rendering policy knobs: run the **performance benchmarks** on a few point counts and both geometries.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hyper3Labs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Hyper3Labs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
