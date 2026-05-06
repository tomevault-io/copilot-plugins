## tracking-module

> PyTorch-based tell-tale tracker for aerodynamics monitoring. Uses fine-tuned RT-DETR detector + custom ByteTracker.

# Tell-Tales Tracking Module

## Overview
PyTorch-based tell-tale tracker for aerodynamics monitoring. Uses fine-tuned RT-DETR detector + custom ByteTracker.

## Key Components
- detector.py: RT-DETR model wrapper for tell-tale detection
- tracker.py + tracker_utils/byte_tracker.py: tracking pipeline
- pipeline.py: Full processing pipeline (detect + track + PCA analysis + classification)
- crop_module/: PCA-based crop analysis, background subtraction (OpenCV MOG2, VPI)
- models/: Pydantic data models (Detection, Track, BoundingBox, Image, Layout, etc.)
- classifyer/: Tell-tale state classification
- video.py: VideoReader, FFmpegVideoWriter (simpler version, no stereo)
- streamer.py: Frame streaming for real-time processing

## Model Checkpoints
- Located at checkpoints/ (project root)
- rt-detr.pt: RT-DETR fine-tuned detector
- yolo-s.pt: YOLO-S alternative detector

## Configuration
- YAML parameter files in parameters/
- Layout definitions in assets/tracking/layouts/

## Testing
- Tests in tests/tracking/
- Run with: make test-tracking
- Fixtures in fixtures/ (project root)

---
> Source: [estebanfoucher/Sail-CV](https://github.com/estebanfoucher/Sail-CV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
