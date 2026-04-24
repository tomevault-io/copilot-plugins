## swervin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Swervin is a vision-based turn assist prototype: a perception-to-control pipeline that estimates lane geometry from a front camera feed and computes a steering recommendation using Pure Pursuit control. It is **not** autonomous driving — it only activates when lane confidence is high and deliberately suppresses itself rather than hallucinating guidance.

## Pipeline Architecture

The system processes frames in this sequence:

1. Camera frame → **Lane Segmentation** (ERFNet recommended; ENet as lightweight fallback)
2. Segmentation mask → **BEV Transform** (OpenCV perspective/IPM transform applied to the mask, not raw image)
3. BEV mask → **Geometry Extraction** (contour/point extraction, polynomial fit, optional RANSAC, centerline generation)
4. Geometry → **Pure Pursuit Controller** (computes desired steering target from centerline + lookahead point)
5. Desired steering vs. actual steering → **Confidence-Gated Visualizer** (disabled when confidence is low)

## Key Design Constraints

- **Segmentation model interface must be swappable** via a unified function signature:
  ```python
  mask, confidence, metadata = segment_lane(frame)
  ```
- **Assist activates only when**: lane mask confidence is high, centerline is stable, visible extent is sufficient, BEV geometry is plausible, curvature is stable across frames.
- **Prefer no guidance over wrong guidance** — if confidence drops, suppress the assist entirely.
- Camera setup is fixed (calibrated mount, fixed resolution/crop, stable intrinsics/extrinsics for BEV).

## MVP Scope

Build these in order:
1. Lane segmentation on real footage
2. BEV transform
3. Centerline geometry extraction
4. Pure Pursuit steering target
5. Confidence-based activation gating
6. Current vs. desired steering visualization

**Not in MVP**: lane change assist, object detection, full 3D lane detection, attention modules (FCA/CBAM/ECA/SE), InSegNet reimplementation.

## Model Notes

- **ERFNet** is the recommended starting model (~8 MB, best precision/speed tradeoff).
- **ENet** is the lightweight fallback (~1.5 MB, lower precision, higher false positives).
- **DeepLabv3+** and **InSegNet-v4** are post-MVP options.
- Preferred pretrained sources: `voldemortX/pytorch-auto-drive`, `cardwing/Codes-for-Lane-Detection`, `Turoad/lanedet`.
- Fine-tune on data from the exact deployment camera after starting from a public pretrained model — domain mismatch is the primary segmentation failure mode.

## Dataset References

- **CurveLanes** — best alignment with turn-assist geometry (100K train)
- **CULane** — robustness benchmark (88K train)
- **LLAMAS** — segmentation-oriented (100K+ annotated)
- **TuSimple** — clean highway baseline for startup validation (6.4K labeled)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CALEBCH0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
