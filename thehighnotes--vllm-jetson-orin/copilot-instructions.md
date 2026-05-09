## vllm-jetson-orin

> Pre-built vLLM 0.17.0 wheel with Marlin GPTQ kernels compiled for SM 8.7 (Jetson Orin family). Includes scheduler fast-path patch, systemd service template, and benchmark scripts.

# vLLM Jetson Orin — Development Notes

## Project Overview

Pre-built vLLM 0.17.0 wheel with Marlin GPTQ kernels compiled for SM 8.7 (Jetson Orin family). Includes scheduler fast-path patch, systemd service template, and benchmark scripts.

See README.md for full documentation.

## Publishing Consideration (noted 2026-04-01)

### PyPI: `vllm-jetson-orin`

Package name is unclaimed on PyPI. The wheel is a platform-specific binary (`linux_aarch64`, `cp310`), which PyPI supports via platform tags — `pip install vllm-jetson-orin` would work on matching Jetson systems and show "no compatible version" elsewhere (normal behavior for platform wheels, same as torch/triton).

**What to publish:** The .whl artifact + scripts/patches/service as package data or entry points.

**What to do:**
- Write a `pyproject.toml` with correct platform tags and metadata
- The dependency chain (PyTorch from `pypi.jetson-ai-lab.io`, CUDA 12.6, JetPack 6.x) cannot be expressed as pip dependencies — document as prerequisites (same pattern as other GPU packages on PyPI)
- Consider entry points for `start_server.sh` and benchmark scripts

**Priority:** High — quickest win, fills a real gap. Only pre-built vLLM with SM 8.7 Marlin kernels anywhere.

---
> Source: [thehighnotes/vllm-jetson-orin](https://github.com/thehighnotes/vllm-jetson-orin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
