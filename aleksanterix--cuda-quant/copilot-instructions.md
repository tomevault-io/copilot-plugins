## cuda-quant

> `cuda-quant` is a **GPU-native quantitative computation library** written in Python using **Numba CUDA**.

# cuda-quant — Project Plan

## 1. Purpose

`cuda-quant` is a **GPU-native quantitative computation library** written in Python using **Numba CUDA**.

Its goal is to provide **technical-analysis-style primitives and indicators** (similar to `ta`, `talib`, etc.) that:
- run **entirely on the GPU**
- avoid Python loops
- avoid NumPy during computation
- are composable, minimal, and explicit

This is **private infrastructure code**, optimized for correctness first, performance second, and extensibility third.

---

## 2. Non-Goals

The following are explicitly **out of scope**:

- Pandas integration inside kernels
- Automatic CPU fallbacks
- Fancy abstractions or class hierarchies
- Dynamic typing or magic dispatch
- UI, plotting, or visualization
- Reinventing CUDA or Numba internals

If it smells like a framework, it does not belong here.

---

## 3. High-Level Architecture

The library is organized into **four strict layers**:

 Indicators->Primitives->Kernels->GPU


### 3.1 Indicators
- Financial meaning only (RSI, EMA, MACD, etc.)
- No kernels
- No memory allocation logic
- Pure function composition

### 3.2 Primitives
- Mathematical building blocks
- Allocate output arrays
- Launch kernels
- No financial logic

### 3.3 Kernels
- Numba CUDA only
- Raw numerical operations
- No allocations
- No Python control flow beyond bounds checks

### 3.4 Core Utilities
- GPU detection
- Thread/block configuration
- Array conversion helpers

---

## 4. Directory Structure

The project follows a clean, layered structure.

```
cuda_quant/
├── core/         # GPU utilities, array helpers
├── kernels/      # Numba CUDA kernels ONLY
├── primitives/   # Kernel launchers & math building blocks
├── indicators/   # Financial indicators (RSI, EMA, etc.)
├── io/           # Data conversion helpers (NumPy, Pandas)
├── benchmarks/   # Performance comparisons
└── tests/        # Correctness tests
```

Each directory has **one responsibility**.

---

## 5. Data Model

### 5.1 Arrays
- All computations operate on **1D GPU arrays**
- Type: `float32` by default
- Input arrays must already be on the GPU or explicitly converted

### 5.2 Shapes
- Initial version assumes shape `(n,)`
- Batched `(symbols, time)` comes later
- No ragged arrays
- No dynamic resizing

---

## 6. Execution Model

1. User converts data to GPU array
2. Indicator function is called
3. Indicator composes primitives
4. Primitives allocate outputs + launch kernels
5. Kernels execute on GPU
6. Result remains on GPU until explicitly copied

---

## 7. Implementation Phases

### Phase 1 — Foundation (Correctness) - COMPLETE
- `as_cuda_array`
- `diff`
- `clip`
- `rolling_mean`
- RSI

### Phase 2 — Core Indicators (Correctness) - COMPLETE
- EMA
- SMA
- MACD
- VWAP

### Phase 2.5 — Extended Indicators (Correctness) - COMPLETE
- DEMA / TEMA
- Bollinger Bands
- ATR (Average True Range)
- Stochastic Oscillator
- OBV (On-Balance Volume)
- CCI (Commodity Channel Index)
- ROC (Rate of Change)

### Phase 3 — Performance (Planned)
- Prefix-sum rolling ops
- Kernel fusion
- Shared memory
- Stream parallelism

### Phase 4 — Scaling (Planned)
- Batched symbols `(symbols, time)`
- Multi-GPU (optional)
- Mixed precision

---

## 8. Testing Strategy

- Tests compare GPU output vs NumPy/Pandas reference
- Small arrays only
- Deterministic seeds
- No performance tests in CI

Correctness > speed.

---

## 9. Performance Philosophy

The library prioritizes:
1. **Avoiding Python**
2. **Avoiding host-device transfers**
3. **Large contiguous workloads**
4. **Simple kernels that can be optimized later**

Premature optimization is discouraged.

---

## 10. Code Ownership Expectations

Every contributor is expected to:
- Understand CUDA execution basics
- Understand GPU memory constraints
- Read kernel code carefully
- Treat performance regressions seriously

This is not beginner code.

# cuda-quant — Engineering Rules

These rules are **non-negotiable**.
Violations will be rejected in review.

---

## 1. General Rules

1. This is a **GPU-first codebase**
2. Explicit is better than clever
3. Readability beats micro-optimizations
4. One responsibility per file
5. No silent behavior

---

## 2. CUDA & Numba Rules

1. **All kernels must use `@cuda.jit`**
2. **Kernels must not allocate memory**
3. **Kernels must not call other kernels**
4. **No Python objects inside kernels**
5. **Bounds checks are mandatory**
6. **One kernel = one mathematical operation**

If you need more logic, write another kernel.

---

## 3. Memory Rules

1. All inputs must be GPU arrays
2. No implicit host ↔ device copies
3. Output arrays are allocated in primitives
4. Indicators must never allocate arrays directly
5. Device synchronization must be explicit

---

## 4. Indicator Rules

1. Indicators are **pure functions**
2. No classes for indicators
3. No kernel launches inside indicators
4. No side effects
5. No state, caching, or mutation

Indicators may only:
- call primitives
- combine results mathematically

---

## 5. Primitive Rules

1. Primitives launch kernels
2. Primitives allocate output arrays
3. Primitives contain **no financial logic**
4. Primitives must be reusable

If logic is indicator-specific, it does not belong here.

---

## 6. Core Utilities Rules

1. `core/` contains infrastructure only
2. No math in `core/`
3. No finance logic in `core/`
4. No kernel code in `core/`

---

## 7. Style Rules

1. Functions > classes
2. Flat code > deep hierarchies
3. No metaprogramming
4. No decorators beyond `@cuda.jit`
5. No hidden dispatch or magic flags

---

## 8. Performance Rules

1. Never loop over data in Python
2. Avoid launching many tiny kernels
3. Favor contiguous memory access
4. Measure before optimizing
5. Never guess performance

Benchmarks must justify optimizations.

---

## 9. Error Handling Rules

1. Fail fast
2. Fail explicitly
3. Do not silently correct user input
4. Validate shapes early

---

## 10. Review Checklist

Before merging, ask:

- Does this stay on the GPU?
- Does this introduce hidden allocations?
- Does this violate layer boundaries?
- Is the kernel minimal and focused?
- Is correctness preserved?

If unsure — reject.

---

## 11. Cultural Rule (Important)

This project values:
- correctness over hype
- simplicity over abstraction
- understanding over copying

If you do not understand a kernel, **do not modify it**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aleksanterix)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aleksanterix)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
