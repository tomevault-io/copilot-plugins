## inference-jetson

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Inference_3 is a radar-based 3D object detection inference pipeline targeting NVIDIA Jetson (JetPack 6) via TensorRT. It handles the full deployment cycle: PyTorch checkpoint → ONNX → TensorRT engine → inference → evaluation.

Input: 3-frame temporal windows of RAD (Range-Azimuth-Doppler) radar cubes
Output: RAE (Range-Azimuth-Elevation) detection cubes compared against LiDAR ground truth

## Commands

### Installation
```bash
uv sync  # preferred on Jetson
# or: pip install -e .
```

### Run the pipeline
```bash
# Build TensorRT engine
python main.py --build --arch FPN --encoder resnet18 --quantization fp16 --ckpt /path/to/model.ckpt

# Run inference
python main.py --infer --arch FPN --encoder resnet18 --quantization fp16

# Evaluate metrics
python main.py --evaluate --arch FPN --encoder resnet18 --quantization fp16

# Benchmark latency
python main.py --measure_infer_time --arch FPN --encoder resnet18 --quantization fp16

# Chain multiple steps
python main.py --build --infer --evaluate --arch FPN --encoder mobilenet_v2 --quantization fp16 --ckpt /path/to/model.ckpt

# Force rebuild even if outputs already exist
python main.py --build --force_build --arch FPN --encoder resnet18 --quantization fp16 --ckpt /path/to/model.ckpt

# Force re-calibration for INT8 (deletes existing cache)
python main.py --build --force_calib --arch FPN --encoder resnet18 --quantization int8 --ckpt /path/to/model.ckpt
```

### Tests
```bash
python test/check_imports.py   # Validate dependencies
python test/check_loader.py    # DataLoader sanity check
```

## Architecture

### Data Flow
```
Input RAD Cube: (1, 3, 2, D=128, R=512, A=256)
  └─ 3 frames × 2 channels (elevation + power)

Per frame: DopplerEncoder (3D Conv) → (B, 64, H, W)
  └─ Cascaded Conv3d reducing Doppler dimension: 128→30→15→1

Per frame: SMP Segmentation Model → (B, 34, H, W)
  └─ FPN or Unet with configurable encoder (resnet18, mobilenet_v2, ...)

Temporal Smoothing: stack 3 frames → (B, 3, 34, H, W)
  └─ Conv3d stack (3→6→12→6→3 channels)

Output RAE Cube: (1, 3, 34, H, W)
```

### Key Constants (`config/config.py`)
- `OUT_CLASSES = 34` — elevation angle classes
- `IN_CHANNELS = 64` — DopplerEncoder output channels

### TensorRT Export
`ExportWrapper` (in `scripts/build.py`) reshapes the batch+time dimensions into a single batch axis (`B×T`) to produce a linear ONNX graph compatible with TensorRT. This avoids complex branching that breaks TRT parsing.

Quantization modes:
- **FP32**: no modification
- **FP16**: `config.set_flag(trt.BuilderFlag.FP16)`
- **INT8**: `RadCubeCalibrator` (entropy calibration, 30 batches from Scene 6)

### Inference Execution (`utils/trt_wrapper.py`)
`TRTInferenceWrapper` uses PyCUDA async execution: H2D copy → async execute → D2H copy → CUDA sync. Requires TensorRT 8.5+ API.

### Dataset (`data/rad_cube_loader.py`)
`RADCUBE_DATASET_TIME` loads 3-frame temporal windows from the RaDelft Dataset.
Default path: `/home/ahernandez/shared/RaDelft-Dataset`
Train scenes: `[1, 3, 4, 5, 7]`, Test scenes: `[2, 6]`

Scene structure: `SceneN/RadarCubes/` (Pow/Ele/Dop frames + timestamps) and `SceneN/rosDS/rslidar_points_clean/` (LiDAR GT).

### Evaluation Metrics
- **Pd** (Probability of Detection): TP / (TP + FN)
- **Pfa** (Probability of False Alarm): FP / (FP + TN)
- **Chamfer Distance**: bidirectional nearest-neighbor distance in meters

Results appended to `results/results_laptop.csv` (x86_64) or `results/results_jetson.csv` (aarch64).

## CLI Reference

| Argument | Values | Notes |
|---|---|---|
| `--arch` | FPN, Unet | required |
| `--encoder` | resnet18, mobilenet_v2, ... | required |
| `--quantization` | fp32, fp16, int8 | default: fp32 |
| `--subsampling` | decim, maxpool, avgpool | optional |
| `--subsample_axes` | R, A, D, RA, RD, RAD | default: RA |
| `--zero_filling` | flag | preserves input shape |
| `--pool_size` / `--pool_stride` | int | default: 2 |
| `--ckpt` | path | required for `--build` |
| `--dataset-path` | path | overrides hardcoded default |
| `--force_build` | flag | force rebuild even if engine exists |
| `--force_calib` | flag | force re-calibration for int8 (deletes cache) |

## Platform Notes

- **Python 3.10** strictly (enforced in `pyproject.toml`)
- **TensorRT** must be installed at OS level (JetPack or manual); not pip-installable
- **Jetson (aarch64)**: torch/torchvision from `pypi.jetson-ai-lab.io/jp6/cu126`
- **x86_64**: torch/torchvision from `https://download.pytorch.org/whl/cu121`
- `numpy < 2` required (compatibility constraint)

---
> Source: [AdrienHrdz/Inference_Jetson](https://github.com/AdrienHrdz/Inference_Jetson) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
