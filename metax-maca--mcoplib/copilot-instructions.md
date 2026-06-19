## mcoplib

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**mcOpLib** is a custom operator library for LLM inference frameworks (vLLM, SGLang, LMDeploy). It provides optimized CUDA kernels for attention mechanisms, MoE (Mixture of Experts) layers, quantization, and other transformer operations. The library is designed for MACA (Metax GPU) architecture but also supports CUDA.

**Key Architecture Points:**
- The project is structured as a Python package (`mcoplib`) with multiple C++/CUDA extension modules
- Each framework has its own submodule: vLLM, SGLang, LMDeploy, plus a default op module
- The build system uses CMake with pybind11 bindings, orchestrated through setup.py
- Extension modules are built conditionally based on environment variables

## Building the Project

### Prerequisites

Always build inside published vLLM/SGLang Docker images. The project requires:
- Python 3.9+
- CMake 3.30.4
- pybind11
- MACA SDK or CUDA toolkit
- torch 2.6.x

### Environment Setup

Before building, source the environment script:
```bash
source env.sh
```

This sets up critical paths like `MACA_PATH`, `CUDA_PATH`, `CUCC_PATH`.

### Compilation Methods

**Full build (no logs shown):**
```bash
pip install -e . --no-build-isolation
```

**Full build (with verbose output):**
```bash
pip install -e . --no-build-isolation -v    # basic verbosity
pip install -e . --no-build-isolation -vv   # more verbose
pip install -e . --no-build-isolation -vvv  # most verbose
```

**Incremental builds (faster for development):**
```bash
python setup.py develop
python setup.py build_ext --inplace  # rebuilds extensions only
```

**Enable WCUDA debug output:**
```bash
export WCUDA_DEBUG=1
```

**Note:** When using `pip install -e .` with verbosity flags, print statements in setup.py are buffered and only appear after completion or failure. Use `python setup.py` directly for immediate output during development.

### Packaging for Distribution

```bash
cd /path/to/source/dir
python -m build --no-isolation
# Output whl is in dist/ directory
```

## Controlling Build Configuration

The build can be controlled via environment variables to selectively compile submodules:

```bash
export BUILD_VLLM_SUBMODULE=OFF         # Disable vLLM ops (default: ON)
export BUILD_SGLANG_SUBMODULE=OFF       # Disable SGLang ops (default: ON)
export BUILD_LMDEPLOY_SUBMODULE=OFF     # Disable LMDeploy ops (default: ON)
export BUILD_DEFAULT_OP_SUBMODULE=OFF   # Disable default ops (default: ON, rarely disable)
```

**Important:** `BUILD_DEFAULT_OP_SUBMODULE` must remain enabled in most cases because:
1. Default operators are reused by other frameworks
2. `import mcoplib` defaults to importing `mcoplib.op`
3. Disabling it will cause import errors

## Directory Structure

```
mcoplib/
├── op/                      # Operator source code
│   ├── vllm/               # vLLM-specific kernels
│   │   ├── attention/      # Paged attention, MLA, etc.
│   │   ├── moe/            # MoE operations (topk, gate, fused ops)
│   │   └── quantization/   # AWQ, GPTQ, GGUF, FP8 kernels
│   ├── sglang/             # SGLang-specific kernels
│   │   └── csrc/           # CUDA/C++ implementations
│   ├── lmdeploy/           # LMDeploy-specific kernels
│   └── *.cu                # Default ops (fused ops, RMS norm, rotary embedding)
├── include/                # Shared header files
├── kernel/                 # Additional kernel utilities
├── CMakeLists.txt          # Main CMake build configuration
├── setup.py                # Python build orchestration
├── env.sh                  # Environment variable setup
└── requirements/           # Python dependencies
```

## Extension Modules

The project builds separate Python extension modules:

1. **`mcoplib.op`** (default ops) - Core fused operations used across frameworks
2. **`mcoplib._C`** (vLLM) - Main vLLM operators
3. **`mcoplib._moe_C`** (vLLM MoE) - MoE-specific operations
4. **`mcoplib.lmdeploy`** (LMDeploy) - LMDeploy operators
5. **`mcoplib.sgl_kernel`** (SGLang) - Core SGLang kernels
6. **`mcoplib.sgl_grouped_gemm_cuda`** (SGLang) - Grouped GEMM operations
7. **`mcoplib.sgl_moe_fused_w4a16`** (SGLang) - Fused W4A16 MoE kernels
8. **`mcoplib.sgl_grouped_gemm_mctlass_int8`** (SGLang) - Int8 grouped GEMM

## CMake Architecture

The build uses a custom `cmake_build_ext` class in setup.py that:
1. Configures all CMake extensions together in a single build directory
2. Builds all extensions with one CMake invocation
3. Installs each extension to its respective destination

Key functions:
- `define_gpu_extension_target()` - Defines CUDA/C++ extension targets with proper flags
- `set_gencode_flags_for_srcs()` - Sets GPU architecture compilation flags
- Custom compilation flags are set per-file for optimization (e.g., GPTQ Marlin kernels)

## GPU Architecture Support

The project targets multiple GPU architectures defined in `CUDA_SUPPORTED_ARCHS`:
- CUDA 12.8+: 7.0, 7.2, 7.5, 8.0, 8.6, 8.7, 8.9, 9.0, 10.0, 10.1, 12.0
- CUDA < 12.8: 7.0, 7.2, 7.5, 8.0, 8.6, 8.7, 8.9, 9.0

## Common Development Workflows

### Adding a New Operator

1. Create `.cu` file in appropriate `op/` subdirectory (vllm/sglang/lmdeploy or default `op/`)
2. Add source file to corresponding `CMakeLists.txt` source list
3. Add pybind11 bindings in the appropriate `.cpp` file
4. Rebuild using `python setup.py build_ext --inplace`

### Modifying Existing Kernels

After making changes to `.cu` or `.cpp` files:
```bash
python setup.py build_ext --inplace
```

This performs an incremental rebuild, only recompiling changed files.

### Running Unit Tests

Python tests are in `unit_test/` directory. C++ tests for CV ops:
```bash
cd unit_test/cpp
mkdir build && cd build
cmake_maca .. && make_maca
```

### Debugging Kernel Traps

To add source code line info for debugging kernel traps:
```bash
export DEBUG_LINE_INFO=1
```

This adds `--generate-line-info` to CUDA compilation flags.

## CV Op Kernel Separate Build

The CV operators have a separate build flow:

```bash
cd op/cv/
cmake_maca -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake_maca --build build -j$(nproc)
cd build
cpack -G DEB  # Generates .deb package
```

Install the CV ops deb package:
```bash
dpkg -i mcoplib_cv-0.2.0-Linux.deb
# Files installed to /opt/maca-ai/mcoplib/
```

## Version Information

After installation, check version with:
```bash
mcoplib_version
```

Version format: `{version}+maca{maca_ver}-torch{torch_major.minor}`
Example: `0.3.1+maca3.0.0.8-torch2.6`

Version info is written to `mcoplib/version` file during build and includes:
- Mcoplib version
- Build MACA version
- Git branch and commit
- vLLM op version
- SGLang op version

## Testing and Benchmarking

- **mxbench** - Automated performance testing for op kernels
- **profiler tools** - Check kernel performance
- **unit_test/** - Python unit tests for operators

## Common Issues

1. **"cmake_maca not found"** - Run `source env.sh` before building
2. **"No module named build.__main__"** - Reinstall build package: `pip install --force-reinstall build`
3. **Version file missing** - Install git command in build environment
4. **Network shared drive compilation** - Install cmake as root user in container

## Key Dependencies

- **CUTLASS/MATLASS** - Template libraries for CUDA/MACA GEMM operations (for vLLM ops)
- **flashinfer** - Flash attention kernels (for SGLang ops)
- **pybind11** - Python/C++ bindings
- **torch.utils.cpp_extension** - CUDA extension utilities

---
> Source: [MetaX-MACA/mcoplib](https://github.com/MetaX-MACA/mcoplib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
