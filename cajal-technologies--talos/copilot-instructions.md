## talos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A project for **verifying WebAssembly code using Lean 4**. The vehicle is a built-in Wasm interpreter written in Lean: the same definitions that _execute_ a program are the ones you _reason about_, so there is no separate "spec" interpreter to keep in sync with the runner.

The interpreter is deliberately optimized for **simplicity of reasoning, not execution speed**. When making changes, prefer the formulation that is easiest to unfold and `simp` through in proofs over the one that runs faster — performance work belongs behind a separate, proven-equivalent implementation, not in the reference interpreter.

Lean toolchain is pinned in `lean-toolchain`.

## Repository layout

Three Lake packages in a monorepo, forming a strict dependency chain:

```
interpreter/   ← Wasm AST, semantics, WP tactic layer  (Lake package: Interpreter)
codelib/       ← lifting lemmas and reasoning helpers   (Lake package: CodeLib)
programs/      ← concrete Rust-to-Wasm verification     (Lake package: Programs)
```

`programs/rust/` holds the Rust source crates; `programs/Programs/` holds the
generated `Program.lean` files and hand-written `Spec.lean` / `Proofs.lean`.

Dependency direction: `interpreter` → `codelib` → `programs`. Downstream code
imports `CodeLib`, never the interpreter directly.

## Build / run / verify

```bash
# from each package directory:
lake build       # builds the libraries and executables
```

There is no separate test runner. Example correctness is encoded as Lean theorems and `native_decide` checks inside the examples; a successful `lake build` means every proof and decidable example check passed. To check a single source file in isolation: `lake env lean <path>`.

## Architecture

Three layers, deliberately small and currently i32-only:

- **Syntax (AST).** Instructions, functions, and modules. The instruction set today is a minimal i32 core (arithmetic, comparisons, locals, structured control flow `block`/`loop`/`br`/`brIf`/`iff`, `call`, `ret`, `drop`). No memory, globals, tables, imports, or non-i32 value types yet.
- **Semantics (interpreter).** A fuel-bounded big-step interpreter built around a single `step : Config → StepResult` function iterated by a `runLoop`. The execution state is a frame holding an operand stack (top-of-stack is the head of the list), a label stack for structured control flow, the remaining instructions, and locals; calls push the caller frame onto a call stack and return merges the callee's operand stack back onto it. Each label carries two continuations — one used on branch, one used on fall-through — which is how `block` vs `loop` differ. Insufficient operands, out-of-bounds access, division by zero, etc. produce a trap, which `run` surfaces as `none`.
- **Reasoning (examples).** Each example pairs a hand-built module with proofs. The standard proof style is: unfold the interpreter and `simp` to reduce both sides to the same concrete computation; concrete-input sanity checks use `native_decide`; larger results compose previously proven theorems as black boxes rather than re-unfolding the interpreter. New examples should follow this pattern.

## Public spec API: don't expose fuel

`run` takes an explicit `fuel : Nat` so that it terminates syntactically, but fuel is a proof obligation, not part of what a wasm function "does". User-facing specs should never mention fuel — no `∃ fuel, run … fuel = some rs` and no fixed numeric fuel in the statement. Use the fuel-free predicates from `Interpreter/Wasm/Spec/Termination.lean` instead:

- `Wasm.TerminatesWith m entry args P` — total correctness (some fuel succeeds, result satisfies `P`). Discharge via `TerminatesWith.of_run` / `of_run_eq` by exhibiting a concrete fuel internally.
- `Wasm.PartiallyMeets m entry args P` — partial correctness (every terminating fuel-bounded run satisfies `P`).

When writing or updating a `@[wasm_spec]` theorem, reach for these — the fuel value belongs inside the proof, not the statement.

## Examples

Examples live in `interpreter/Interpreter/Wasm/Examples/`. Each file defines a hand-built Wasm module and proves theorems about it using the WP tactic layer. The standard pattern: state the property, apply `wp_run` to reduce to a concrete computation, then close with `simp` / `omega` / domain lemmas. New examples should follow this pattern.

`Interpreter/Wasm/Examples/Factorial.lean` is a good reference: it proves a full loop invariant correctness theorem with a WP-based loop rule.

---
> Source: [cajal-technologies/talos](https://github.com/cajal-technologies/talos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
