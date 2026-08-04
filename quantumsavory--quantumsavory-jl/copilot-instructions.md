## quantumsavory-jl

> This folder contains the BenchmarkTools suite for `QuantumSavory.jl`.

# Benchmark Folder

This folder contains the BenchmarkTools suite for `QuantumSavory.jl`.

Organization:
- `benchmarks.jl` is the entrypoint (shared imports/setup + `include(...)` calls).
- `benchmark_*.jl` files contain benchmark definitions grouped by top-level `SUITE` key.
- Keep related benchmarks together and add short comments when introducing a new subgroup.

Conventions:
- For mutating benchmarks (`querydelete!`, `untag!`, etc.), use `setup` with `deepcopy(...)` and `evals=1`.
- Add new benchmarks without deleting existing coverage unless explicitly requested.

---
> Source: [QuantumSavory/QuantumSavory.jl](https://github.com/QuantumSavory/QuantumSavory.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
