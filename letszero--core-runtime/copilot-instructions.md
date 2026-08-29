## core-runtime

> This file is the root contract. Every AI session working in this repo loads it first and obeys it without exception. If a rule below conflicts with a prompt, **stop and ask** — do not silently override.

# Core Runtime — Inviolable Rules

This file is the root contract. Every AI session working in this repo loads it first and obeys it without exception. If a rule below conflicts with a prompt, **stop and ask** — do not silently override.

## The freeze rule

> If it can be expressed in Zero without losing performance or correctness, it does NOT belong here.

Before adding anything to this repo, prove it fails that test. High-level ops (softmax, layernorm, attention), data structures (List, Dict, class), autograd, and quantization helpers all belong in the compiler or the Zero stdlib — never here.

## Code rules

- **Header-only, C++20, no external dependencies.** Standard library only.
- **No virtual methods, no inheritance, no RTTI, no exceptions.** Primitives are POD-like.
- **No heap allocation in metadata.** `Tensor` shape/strides use fixed `std::array`, not `std::vector`.
- **`MAX_DIMS = 8` is permanent.** Never raise it.
- **All public functions are `noexcept` or return `Status`.** Pick one per function and stay consistent.
- **Caller-allocated outputs.** Ops take `Tensor& output`, do not allocate internally.
- **Every op takes `Stream* stream = nullptr` as the final parameter.** Even when CPU ignores it.

## Type system rules

- **`DType` enum values are append-only.** Never reorder, never remove, never reuse a value.
- **Every `switch(DType)` must be exhaustive.** Compile with `-Werror=switch`.
- **Dtype is orthogonal to Tensor.** Do not template `Tensor` on dtype.

## API rules

- **No convenience overloads.** One name, one signature, one behavior. `gather` not `gather`+`index_select`.
- **No hidden state.** No globals except `RuntimeConfig`. RNG state is passed as a `Generator` value.
- **No algorithms in the runtime — only state and access patterns.** PRNG algorithms, activation math beyond the existing set, optimizers — all belong in Zero stdlib.
- **The C API in `c_api.h` (when written) is the freeze contract.** C++ headers are reference implementations.

## Spec-driven workflow

- **Every change starts with a spec in `docs/specs/NNN-*.md`.** No spec, no code.
- **Tests are written from the spec, before the implementation.** Not derived from the code.
- **Do not modify the spec to make the implementation pass.** If they disagree, stop and ask.
- **Do not add anything outside the spec's "API surface" section.** "While I was here…" is forbidden.
- **The "Out of scope" section is binding.** If a prompt asks for something listed there, stop and ask.

## When in doubt

Ask. Do not guess at API shape, do not invent dtypes, do not add ops "for completeness," do not refactor adjacent code. This repo is frozen substrate — the cost of a wrong addition is permanent.

---
> Source: [LetsZero/core-runtime](https://github.com/LetsZero/core-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
