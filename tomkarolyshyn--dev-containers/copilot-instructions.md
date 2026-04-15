## dev-containers

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of Docker container definitions for hardware/software co-design development, targeting EDA tooling, LLVM compiler infrastructure, RTL simulation, and AMD/Xilinx FPGA development. Images are published to DockerHub under `tomkarolyshyn/`.

## Build Commands

```bash
# Build individual images
make llvm-python-build
make rtl-sim-build
make llvm-oss-build
make llvm-verilator-build
make llvm-cuda-build
make vitis-build             # requires pre-downloaded installer (~85GB)

# Run containers interactively
make <image>-run             # e.g., make llvm-python-run

# Smoke tests
make <image>-test            # e.g., make llvm-python-test

# Build all, push all public images, or both
make all
make push
make build-and-push
```

## Architecture

### Image Hierarchy

```
ubuntu:24.04
  ├── llvm-python          (LLVM + Python 3.12 + uv + pre-commit)
  │     ├── llvm-verilator (+ Verilator, built from source)
  │     └── llvm-oss       (+ OSS-CAD-Suite: yosys, verilator, iverilog, etc.)
  └── rtl-sim              (standalone Verilator + cocotb, no LLVM)

nvidia/cuda → llvm-cuda-24  (CUDA + OSS-CAD-Suite + LLVM, self-contained)

ubuntu:24.04 → vitis        (AMD Xilinx Vitis/Vivado 2025.2, local only, not pushed)
```

### Key Patterns

- **Version pins**: All tool versions are centralized in `.env` (loaded by both Makefile and compose.yaml). Edit `.env` to bump versions.
- **Parameterized base images**: Every Dockerfile uses `ARG BASE_IMAGE`, allowing the same Dockerfile to produce different layered images (e.g., `rtl-sim/Dockerfile` builds both `rtl-sim` and `llvm-verilator`).
- **Vitis bind-mount build**: The large AMD installer is `bind`-mounted at build time (`RUN --mount=type=bind`) to avoid bloating image layers. Installer directories are gitignored.
- **Tag scheme**: Tags encode version info (e.g., `3.12-20` for llvm-python, `20-v5.040` for llvm-verilator). All images also get a `latest` tag.
- **WORKDIR**: Most images use `/project`; `oss` uses `/app`.

### Container Definitions

| Directory | compose.yaml service | Base | Key tools |
|-----------|---------------------|------|-----------|
| `llvm-python/` | `llvm-python` | ubuntu:24.04 | LLVM, clang, Python 3.12, uv |
| `rtl-sim/` | `rtl-sim`, `llvm-verilator` | ubuntu:24.04 or llvm-python | Verilator (from source), cocotb |
| `oss/` | `llvm-oss` | llvm-python | OSS-CAD-Suite (yosys, verilator, iverilog) |
| `llvm-cuda/` | `llvm-cuda-24` | nvidia/cuda | CUDA, LLVM, OSS-CAD-Suite |
| `vitis/` | `vitis` | ubuntu:24.04 | Vivado, Vitis 2025.2 |

## Notes

- No CI/CD; builds and pushes are manual via `make`.
- `llvm-cuda` build/run requires Nvidia Container Toolkit for GPU access.
- `vitis` is never pushed to DockerHub (62+ GB, licensing restrictions).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomkarolyshyn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tomkarolyshyn)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
