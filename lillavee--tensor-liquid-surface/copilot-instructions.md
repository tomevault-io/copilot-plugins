## tensor-liquid-surface

> When working **in this repository / folder**, the project root is the directory that contains:

# Agent note — Livewire Tensor Liquid Surface

## Project root (this surface)

When working **in this repository / folder**, the project root is the directory that contains:

- `.tensor_surface_root` (marker)
- `livewire/`
- `DOCS/`
- `API/` (public Python package `tensor_surface`)
- `LICENSE`, `NOTICE`, `licenses/`
- vendored nests: `Livewire Vector Liquid Surface/`, `Livewire Arithmetic Surface/`, `Livefire Floating Surface/`

Resolve it with `livewire.exclusive_tu.project_root.project_root()` or set `WET_TENSOR_ROOT`.
Writable data may be redirected with `WET_TENSOR_DATA`.

## Not this project root

These are **different** roots / systems. Do not treat them as the Tensor Surface root:

- Parent Wet monorepo folder (if this clone sits inside one)
- `Livewire Central Liquid Surface` (Central Liquid Surface / morph ingress)
- Memory Surface / other Livewire * Liquid Surface siblings outside this tree
- `Experimental` lab trees (except when rewriting pointers that still hardcode paths)

## Path portability

Prefer `livewire.exclusive_tu.project_root.project_root()` and root-relative paths.
Do **not** hardcode machine-specific absolute paths.
Debug logs belong in the OS temp directory (`project_root.debug_log_path`), not in the repo.

## Vendored nests

Default Vector bind looks for `Livewire Vector Liquid Surface/` **inside this project root**
(override with `WET_VECTOR_ROOT`). Arithmetic and Floating mirrors are vendored for structure
parity / twin documentation; Vector's nested ALU/FPU clones are self-contained.

## Production vs dev

- **Dev:** this clone; editable `API/` install (`pip install -e API`).
- **Production install:** `%LOCALAPPDATA%\Wet\TensorSurface\` via `INSTALL.cmd` (dev tree stays as-is).

## Owned dependencies (not Wet-wide)

- `owned/ternary_alu` + `wheels/ternary_alu-*.whl` — Harford verdict calculator (prebuilt **Windows cp312** `.pyd` today; Rust/C sources were not present under Arithmetic Surface at vendoring time).
- Native GEMM: `livewire/exclusive_tu/compiled_gemm/native` (`tu_gemm`) — prebuilt **win_amd64 cp312** wheel under `dist/` when present.
- One-click production install: project-root `INSTALL.cmd`.

## Version

Product / API version: **1.0.0**

---
> Source: [LillaVee/Tensor-Liquid-Surface](https://github.com/LillaVee/Tensor-Liquid-Surface) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
