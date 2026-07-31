## flash-attention3-wheels

> Pre-built wheels for [Flash Attention 3](https://github.com/Dao-AILab/flash-attention/tree/main/hopper) — Linux/Windows/Arm(CUDA SBSA).

# Flash-Attention 3 Wheels

Pre-built wheels for [Flash Attention 3](https://github.com/Dao-AILab/flash-attention/tree/main/hopper) — Linux/Windows/Arm(CUDA SBSA).

- **Repo**: windreamer/flash-attention3-wheels (main)
- **License**: Apache-2.0
- **Languages**: Python, Shell, PowerShell, Jinja
- **Pages**: https://windreamer.github.io/flash-attention3-wheels/

## Structure

```
scripts/
  build_wheel.sh        # Linux build (args: CUDA_VER TORCH_VER MAX_JOBS)
  build_wheel.ps1       # Windows build
  generate_matrix.py    # CI matrix generator (queries Docker Hub + PyTorch index)
  generate_pages.py     # GitHub Pages index generator (Jinja2)
  verify_wheel_exports.py
  *.patch               # Build patches (cuda_h_alignment, cutlass_alignment, windows_fix)
templates/              # Jinja2 HTML templates for index pages
.github/workflows/
  build_wheels.yml      # Main CI: biweekly + manual dispatch
  build_wheels_test.yml # Test workflow
  update-pages.yml      # Pages deployment
```

## Build & Release

- **CI**: GitHub Actions, biweekly (every 2nd/4th Sun 22:00 UTC) + `workflow_dispatch`
- **Matrix**: CUDA {12.8,13.0,13.2} × PyTorch {2.8.0,2.12.0} × {linux, windows, arm}
- **Process**: Clone upstream flash-attention → build wheel → rename with date/cuda/torch/abi hash → upload as GitHub Release
- **Wheel naming**: `flash_attn_3-{ver}+{date}+cu{cuda}torch{torch}cxx11abi{abi}+{hash}-{platform}.whl`
- **Patches applied**: cuda_h_alignment_fix, cutlass_alignment_fix, windows_fix (conditional)

## Key Scripts

- `scripts/build_wheel.sh` — sets up CUDA env, installs torch, clones upstream, builds, renames wheel with `change-wheel-version`
- `scripts/generate_matrix.py` — env `MATRIX_TARGET` (linux/windows/arm), `CUDA_VERSIONS`, `TORCH_VERSIONS`; outputs GitHub Actions matrix JSON
- `scripts/generate_pages.py` — fetches releases via GitHub API, generates pip `--find-links` index pages

## CI Optimization

- **Windows CUDA install** (Jimver/cuda-toolkit): uses `method: 'network'` + version-specific `sub-packages` to download only necessary components instead of the full toolkit
- **Linux/Arm builds**: use Docker images (`nvidia/cuda:XXX-devel-ubuntu22.04`) which already contain full CUDA; no Jimver/cuda-toolkit involved
- **Sub-packages vary by CUDA version** (critical — wrong list causes build failure):
  - On Windows, `_dev` variants (e.g. `cusparse_dev`) provide headers + import libs; runtime variants (e.g. `cusparse`) provide only DLLs. For C++ extension builds, only `_dev` packages are needed — runtime DLLs come from PyTorch's pip packages.
  - **CUDA 13.0+**: `["crt", "nvcc", "cudart", "thrust", "nvvm", "cuxxfilt", "cusparse_dev", "cublas_dev", "cusolver_dev"]`
    - `crt` — Compiler tools (cicc, etc.) split from nvcc in 13.0
    - `nvvm` — Compiler IR (libdevice, etc.) split from nvcc in 13.0
    - `nvcc` — CUDA compiler (no longer includes crt/nvvm)
    - `cudart` — CUDA runtime libs + headers (including crt/host_config.h)
    - `thrust` — CCCL/Thrust headers
    - `cuxxfilt` — CUDA demangler
    - `cusparse_dev` — cuSPARSE headers (required by PyTorch's CUDAContextLight.h)
    - `cublas_dev` — cuBLAS headers (cublas_v2.h, cublasLt.h required by PyTorch)
    - `cusolver_dev` — cuSOLVER headers (cusolverDn.h required by PyTorch)
  - **CUDA 12.x**: `["nvcc", "cudart", "thrust", "cuxxfilt", "cusparse_dev", "cublas_dev", "cusolver_dev"]`
    - No `crt` or `nvvm` sub-packages exist — their contents are bundled inside `nvcc`
    - `cudart` includes CRT headers (crt/host_config.h, crt/host_defines.h)
    - `nvcc` includes cicc, ptxas, libdevice internally
    - `cusparse_dev` / `cublas_dev` / `cusolver_dev` — headers for CUDA math libraries

## Install

```bash
pip install flash_attn_3 --find-links https://windreamer.github.io/flash-attention3-wheels/cu128_torch280
```

---
> Source: [windreamer/flash-attention3-wheels](https://github.com/windreamer/flash-attention3-wheels) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
