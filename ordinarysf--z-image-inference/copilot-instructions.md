## z-image-inference

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Z-Image-Inference is a Python project for running the Tongyi-MAI/Z-Image-Turbo diffusion model locally on Apple Silicon using the `diffusers` library.

## Commands

```bash
# Install dependencies
uv sync

# Run image generation
uv run python main.py
```

## Technical Notes

- **Target device**: Apple MPS (Metal Performance Shaders)
- **Model**: `Tongyi-MAI/Z-Image-Turbo` from HuggingFace
- **Precision**: Uses `float16` (bfloat16 may have compatibility issues with MPS)
- **Pipeline**: `ZImagePipeline` from diffusers with `low_cpu_mem_usage=False` for MPS compatibility

---
> Source: [OrdinarySF/z-image-inference](https://github.com/OrdinarySF/z-image-inference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
