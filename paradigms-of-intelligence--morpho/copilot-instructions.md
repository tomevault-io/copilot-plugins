## morpho

> Welcome, AI coding assistant! To maintain the highest quality, performance, and readability in this repository, follow these core development rules.

# Agent Coding Guidelines

Welcome, AI coding assistant! To maintain the highest quality, performance, and readability in this repository, follow these core development rules.

> [!IMPORTANT]
> **Core Objective**: Always write code that is as simple, brief, and clear as possible. Prefer short, self-contained functions and flat structures over deeply nested ones.

---

## 1. Simplicity & Brevity First
* **Guard Clauses**: Avoid deeply nested `if/else` structures. Check conditions early and return early.
* **Keep Code Short**: Write concise functions that do one thing. If a block of code gets long or repetitive, extract it into a focused private helper.
* **No Speculative Code**: Avoid writing code for hypothetical future requirements. Only implement what is requested or strictly required.

## 2. Structural & Layout Guidelines
* **Struct-of-Arrays (SoA)**: The engine uses SoA (Flat Typed Arrays) for high-performance memory layout (cells, nets, pins, layouts). Ensure SoA integrity is maintained when adding or modifying entities.
* **No Unnecessary State**: Objects should remain as stateless as possible between transactions. State that can be computed or passed on-demand should not be persisted on instances.
* **Lazy Synchronization**: In performance-sensitive subsystems (like the `LayoutEngine`), decouple updates from actions. Use dirty flags to run updates lazily.

## 3. Systematic Testing & Benchmarking
* **Oracles & Invariants**: Avoid fragile hardcoded verification checks. Write generalized properties (e.g. Circular Linkages, Fanout conservation) and mathematical assertions (e.g. `a * b`) to test correctness under full sweeps or randomized fuzzing.
* **Benchmarks**: Keep performance benchmarks isolated from correctness assertions. Ensure JIT compiler warmup is performed before measuring elapsed timings.

---
> Source: [paradigms-of-intelligence/morpho](https://github.com/paradigms-of-intelligence/morpho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
