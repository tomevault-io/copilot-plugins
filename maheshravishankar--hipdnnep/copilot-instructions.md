## hipdnnep

> This is an out-of-tree ONNXRuntime Execution Provider (EP) that uses AMD's

# Codex Instructions for hipDNNEP

This is an out-of-tree ONNXRuntime Execution Provider (EP) that uses AMD's
hipDNN library for accelerated inference on AMD GPUs. The EP is built as a
plugin that can be dynamically loaded by ONNXRuntime.

## Primary Repo Guidance

- Supported ops:
  - `Conv2D` via hipDNN graph API
  - `MatMul/Gemm` via hipBLAS-LT when enabled
- Optional features:
  - hipBLAS-LT support
  - Torch-MLIR integration via `HIPDNN_EP_ENABLE_TORCH_MLIR=ON`
- Build environment:
  - `THEROCK_DIST=/path/to/TheRock/build/dist/rocm`
  - `ONNXRUNTIME_ROOT=/path/to/onnxruntime`

## Build Commands

### Standard Build

```bash
cmake --preset RelWithDebInfo
cmake --build --preset RelWithDebInfo
ctest --preset RelWithDebInfo --output-on-failure
```

### Build With Torch-MLIR

```bash
cmake --preset RelWithDebInfo-MLIR
cmake --build --preset RelWithDebInfo-MLIR
ctest --preset RelWithDebInfo-MLIR --output-on-failure
```

One-time torch-mlir build:

```bash
cd third_party/torch-mlir
cmake -G Ninja -B ../../../build/torch-mlir \
  -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DCMAKE_INSTALL_PREFIX=../../../build/torch-mlir/install \
  -DLLVM_ENABLE_PROJECTS=mlir \
  -DLLVM_TARGETS_TO_BUILD=host \
  -DMLIR_ENABLE_BINDINGS_PYTHON=OFF \
  -DTORCH_MLIR_ENABLE_STABLEHLO=OFF \
  -DTORCH_MLIR_ENABLE_REFBACKEND=OFF \
  -DTORCH_MLIR_USE_INSTALLED_PYTORCH=OFF \
  -DTORCH_MLIR_ENABLE_PYTORCH_EXTENSIONS=OFF \
  externals/llvm-project/llvm
cmake --build ../../../build/torch-mlir --target install
cd ../..
```

## Code Style

- Use clang-format with the repo `.clang-format`.
- Prefer `static` helpers in anonymous namespaces for file-local logic.
- Keep implementation details out of headers when possible.
- Use RAII and smart pointers for resource ownership.
- Prefer early returns over nested validation branches.

## Testing

- Runtime tests use Google Test.
- Torch-MLIR pass tests use lit + FileCheck.
- When adding lit tests:
  - Align `CHECK:` colons vertically.
  - Capture SSA values as `%[[NAME:.*]]`, not hardcoded `%0`.
  - Use `CHECK-SAME` to split long signatures.

## Worktree And Bead Workflow

Use the beads tracker as the durable task log and source of truth:

- Claim and update work with `br`.
- Create one git worktree per bead.
- Do not implement features directly on the main checkout.
- Record decisions, PR links, and review outcomes in bead comments so the
  workflow does not depend on private chat memory.

Recommended branch pattern:

- `users/<author>/<branchName>`
- Example: `users/MaheshRavishankar/addClangFormat`

## Durable Memory In Codex

Claude's agentic-memory does not have a direct repo-local equivalent here.
For Codex, treat these as persistent memory:

- `br show`, `br comments`, and bead status
- git history and worktree names
- PR descriptions and review comments
- repo docs under `docs/`

If a decision matters later, write it down in one of those places.

## Codex Workflow

Codex-specific reusable workflows live in the repo-local plugin:

- `plugins/hipdnnep-codex/`

That plugin provides skills for:

- `implementer`
- `pm`
- `review`

Use the Codex sandbox launcher for isolated implementer work:

```bash
scripts/hipdnn-codex-sandbox.sh <bead-id> -- "<codex-args...>"
```

The Claude workflow remains available through `scripts/hipdnn-sandbox.sh`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MaheshRavishankar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MaheshRavishankar)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
