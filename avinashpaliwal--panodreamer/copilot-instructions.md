## panodreamer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PanoDreamer converts a single input image into a full 360° 3D scene. It is a research implementation of the SIGGRAPH Asia 2025 paper "PanoDreamer: Optimization-Based Single Image to 360° 3D Scene With Diffusion" by Paliwal et al.

## Setup

```bash
# Clone with submodules (3d-moments for inpainting)
git clone --recursive <repo-url> && cd panodreamer

# Python environment (uses uv)
uv venv && source .venv/bin/activate && uv pip install -e .

# External dependencies
git clone https://github.com/DepthAnything/Depth-Anything-V2.git
mkdir -p checkpoints
wget -P checkpoints https://huggingface.co/depth-anything/Depth-Anything-V2-Large/resolve/main/depth_anything_v2_vitl.pth
python download_inpainting_ckpts.py  # downloads inpainting model weights via gdown
```

## Pipeline

The system is a sequential 5-stage pipeline. Each stage is a standalone script:

```
Input Image + Prompt
    → [1] multicondiffusion_panorama.py   → 360° panorama (equirectangular)
    → [2] depth_estimation.py             → consistent depth map (.npy)
    → [3] ldi_generation.py              → layered depth images (RGBA + depth per layer)
    → [4] train_gsplat.py                → optimized 3DGS scene (.ply)
    → [5] render_gsplat.py               → rendered video frames
```

There is also `multicondiffusion.py` for perspective (non-panoramic) wide image generation.

### Running the full pipeline

```bash
# Stage 1: Panorama generation
python multicondiffusion_panorama.py --prompt_file examples/29_real_campus_3.txt \
  --input_image examples/29_real_campus_3.png --output_dir output

# Stage 2: Depth estimation
python depth_estimation.py --input_image output/final_output.png \
  --output_dir output_depth --mode panorama

# Stage 3: LDI generation
python ldi_generation.py --input_image output/final_output.png \
  --input_depth output_depth/depth.npy --output_dir output_ldi --num_layers 4

# Stage 4: 3DGS optimization
python train_gsplat.py --ldi_dir output_ldi --output scene.ply --num_iterations 300

# Stage 5: Rendering
python render_gsplat.py --ply scene.ply --output renders --num_frames 720
```

## Architecture

### Core pipeline scripts (top-level)

- **multicondiffusion.py / multicondiffusion_panorama.py** — Image extension via iterative inpainting-conditioned MultiDiffusion. The panorama variant uses cylindrical projection for seamless 360° wraparound.
- **depth_estimation.py** — Extracts overlapping views from the panorama, runs Depth Anything V2 on each, aligns via robust piecewise regression (`ropwr`), and stitches into a consistent depth map. Supports "wide" (perspective) and "panorama" (cylindrical) modes.
- **ldi_generation.py** — Splits depth into bins via AgglomerativeClustering, then inpaints occluded background for each layer using 3d-moments inpainting networks.
- **train_gsplat.py** — Initializes 3D Gaussians from LDI layers, optimizes with 240 evenly-rotated camera poses using L1 + SSIM + L2 depth loss. Standard 3DGS densification/pruning.
- **render_gsplat.py** — Renders circular camera paths through the optimized scene, outputs RGB/depth/alpha frames.

### Supporting modules

- **arguments.py** — `GSParams` (3DGS hyperparameters) and `CameraParams` (focal length, FOV, intrinsic matrix K).
- **gs_init.py** — 3DGS initialization and optimization logic with depth constraints.
- **utils/depth.py** — Depth map colorization/visualization.
- **utils/depth_layering.py** — `get_depth_bins()` for depth quantization into LDI layers.
- **utils/depth_utilsv2.py** — Depth Anything V2 wrapper with lazy model loading.

### External dependencies (checked into repo)

- **3d-moments/** (git submodule) — Google Research inpainting networks, used by `ldi_generation.py` via `core.inpainter.Inpainter`.
- **Depth-Anything-V2/** (cloned repo) — Monocular depth estimation model, used by `utils/depth_utilsv2.py`.

## Key technical details

- Coordinate systems use cylindrical projection for panorama handling (perspective ↔ cylindrical transforms in depth_estimation.py and multicondiffusion_panorama.py).
- Depth alignment across views uses `ropwr` (Robust Piecewise Regression) for consistency.
- 3DGS training loss: `L1 + 0.2 * (1 - SSIM) + depth_weight * L2_depth`.
- 3DGS densification runs from iteration 500–2500 with opacity resets at intervals.
- The `gsplat` library (from Nerfstudio) handles Gaussian rasterization.

## Build system

Uses `pyproject.toml` with `uv` as package manager. No test suite, linter config, or CI pipeline exists — this is a research codebase.

---
> Source: [avinashpaliwal/PanoDreamer](https://github.com/avinashpaliwal/PanoDreamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
