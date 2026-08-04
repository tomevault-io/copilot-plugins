## tiny-tpu

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

**Always read `CONTEXT.md` before any task.**

---

## Project Summary

TinyTPU is a real, synthesizable 4×4 weight-stationary systolic array in SystemVerilog, compiled to WASM via Verilator + Emscripten, and visualized live in the browser. Nothing on screen is faked - the React UI reads state directly out of the compiled RTL.

Pipeline: `rtl/*.sv` → Verilator C++ → Emscripten → `web/public/tiny_tpu.wasm` → React island → SVG animation.

---

## Commands

All RTL/WASM commands run inside **WSL2 Ubuntu**, not Windows PowerShell. Activate the Python venv first:

```bash
source ~/.venvs/tinytpu/bin/activate
```

### RTL Lint
```bash
verilator --lint-only -Wall rtl/*.sv
```

### Simulation (cocotb - run from `sim/`)
```bash
# Golden model tests
pytest sim/golden.py -q

# Individual test suites (run from project root)
cd sim && make MODULE=test_pe TOPLEVEL=pe
cd sim && make MODULE=test_systolic_array TOPLEVEL=systolic_array \
         VERILOG_SOURCES="../rtl/pe.sv ../rtl/systolic_array.sv"
cd sim && make MODULE=test_top TOPLEVEL=tiny_tpu_top \
         VERILOG_SOURCES="../rtl/pe.sv ../rtl/systolic_array.sv ../rtl/controller.sv ../rtl/tiny_tpu_top.sv"
```

### WASM Build
```bash
bash wasm/build.sh
# outputs web/public/tiny_tpu.mjs + web/public/tiny_tpu.wasm
```

### Frontend (from `web/`)
```bash
pnpm dev          # dev server
pnpm build        # production build
pnpm lint         # eslint
pnpm typecheck    # astro check && tsc --noEmit
pnpm format       # prettier
```

### Full pre-PR check
```bash
verilator --lint-only -Wall rtl/*.sv
cd sim && pytest golden.py -q && make MODULE=test_top TOPLEVEL=tiny_tpu_top \
  VERILOG_SOURCES="../rtl/pe.sv ../rtl/systolic_array.sv ../rtl/controller.sv ../rtl/tiny_tpu_top.sv"
cd web && pnpm lint && pnpm typecheck && pnpm build
```

---

## Architecture

### RTL (`rtl/`)

| File | Role |
|---|---|
| `pe.sv` | Single MAC cell. Registered: `psum_out <= psum_in + weight_reg * act_in`, `act_out <= act_in`. |
| `systolic_array.sv` | 4×4 generate-loop of PEs. Horizontal: activations flow right. Vertical: partial sums flow down. |
| `controller.sv` | FSM: `IDLE → LOAD_WEIGHTS → STREAM → DRAIN → DONE`. Applies the diagonal row-skew to matrix A during `STREAM`. |
| `tiny_tpu_top.sv` | Top-level wrapper. Owns input buffers for A/B, the result buffer C, and the **debug output bus** (the only sanctioned way to expose PE state to the C++ harness). |

**Dataflow:** Matrix B is loaded as stationary weights (PE[i][j] = B[i][j]). Matrix A streams in from the left edge, row-skewed - row `i` is delayed `i` cycles so operands meet at the right PE at the right time. Results exit the bottom edge, skewed. Fixed-point: int8 inputs, 32-bit accumulators.

### WASM Bridge (`wasm/`)

- `harness.cpp` - owns a `Vtiny_tpu_top` instance. `step()` advances one clock and reads the debug output bus to produce a `CycleState` object.
- `bindings.cpp` - `EMSCRIPTEN_BINDINGS` exposing `TinyTpuSim` to JS via embind.
- `build.sh` - runs `verilator --cc` then `em++`. Outputs land in `web/public/`.

### State Contract

`docs/STATE_SCHEMA.md` and `web/src/lib/state-schema.ts` define the per-cycle JSON schema shared between the C++ harness and React. These two files **must stay in sync**. `actIn` for PE[i][j] is derived in the harness as `(j==0) ? westInputs[i] : PE[i][j-1].act_out` - it is not a direct register read.

### Frontend (`web/`)

Astro + React islands + shadcn/ui + Tailwind + TypeScript strict. WASM is loaded only inside a `client:only="react"` island via `web/src/lib/wasm-loader.ts` - never during SSR. Key components: `Visualizer` (the island wrapper), `PEGrid` (pure SVG presentational), `Controls` + `useTinyTpu` hook (owns the `CycleState[]` and playback), `MatrixInput`.

---

## Branching Strategy

- `main` is protected - **never commit directly to main**.
- `dev` is the integration branch - all feature/fix/spike branches PR into `dev`.
- `main` is updated only via PRs from `dev`.
- Branch naming: `feature/`, `fix/`, `spike/`, `chore/`, etc. always branch off `dev`.

---

## Non-Negotiable Rules

1. **RTL is the single source of truth.** Never reimplement the matmul in JS/TS for animation. Always read state from the compiled WASM.
2. **RTL must stay synthesizable.** `always_ff`/`always_comb` only. No `#delays`, no `initial` blocks in design modules, no inferred latches, no `$display`.
3. **Every RTL change must bit-match `sim/golden.py` (numpy).** A wrong matmul means a beautiful lie - do not proceed until it matches.
4. **Expose PE state via the top-level debug output bus only.** Do not reach into internal signals. `public_flat` is a last resort.
5. **WASM never loads during SSR.** `client:only="react"` on every island that touches WASM. Import inside `useEffect`, never at module top level.
6. **TypeScript strict, no `any`.** `noUncheckedIndexedAccess: true`.
7. **Conventional Commits** on every commit: `feat:`, `fix:`, `build:`, `ci:`, `test:`, `docs:`, `chore:`, `refactor:`, `perf:`.

---

## Common Failure Modes

| Symptom | Fix |
|---|---|
| `bad interpreter: /bin/bash^M` | CRLF in shell script - `dos2unix file.sh` |
| `UNOPTFLAT` | Combinational loop - ensure passthrough regs use `<=` in `always_ff` |
| `BLKANDNBLK` | Mixed `=`/`<=` in one block - use `<=` in `always_ff`, `=` in `always_comb` |
| Matmul off by one row/col | Skew timing off - adjust per-row activation delay in `STREAM` |
| Result all zeros | Weights not loaded or FSM overlap between `LOAD_WEIGHTS`/`STREAM` |
| cocotb signed compare wrong | Use `.value.signed_integer`, not `.value.integer` |
| `Vtop.h: No such file` | `--Mdir` path ≠ `-I` path in `build.sh` - keep them identical |
| `window is not defined` at build | WASM imported at SSR time - move to `useEffect`/`client:only` |
| `.wasm` 404 | Artifacts must be in `web/public/`; reference as `/tiny_tpu.mjs` |
| Blank prod page | SSR touched WASM - confirm `client:only="react"` everywhere |

---

## PR Checklist

- [ ] `verilator --lint-only -Wall rtl/*.sv` - clean (no warnings)
- [ ] cocotb tests pass; RTL bit-matches `golden.py`
- [ ] No UNOPTFLAT / BLKANDNBLK / inferred latch warnings
- [ ] `pnpm lint && pnpm typecheck && pnpm build` - all clean
- [ ] WASM rebuilt and visualizer still loads if RTL changed
- [ ] No large binaries committed except `web/public/tiny_tpu.wasm` (intentional)
- [ ] Conventional Commit message
- [ ] README/docs updated if behavior changed

---
> Source: [deaneeth/tiny-tpu](https://github.com/deaneeth/tiny-tpu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
