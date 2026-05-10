## realtime-stt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This project aims to deploy Voxtral Mini Realtime 2602 ONNX model with CUDA GPU acceleration for real-time speech-to-text transcription, integrated with LiveKit for voice AI applications. Key goals:
- Optimize GPU utilization for streaming inference
- Support mixed zh-TW/English speech transcription
- Benchmark performance (latency, throughput, accuracy)

## Model Reference

- **Model**: [onnx-community/Voxtral-Mini-4B-Realtime-2602-ONNX](https://huggingface.co/onnx-community/Voxtral-Mini-4B-Realtime-2602-ONNX)
- **Architecture**: Voxtral Realtime (streaming ASR, natively streaming)
- **Languages**: en, fr, es, de, ru, zh, ja, it, pt, nl, ar, hi, ko
- **Paper**: arxiv:2602.11298 - Voxtral Realtime achieves Whisper-quality at sub-second latency with 480ms delay
- **License**: Apache 2.0

## ONNX Runtime GPU Setup

Install with CUDA 12.x support:
```bash
pip install onnxruntime-gpu
```

Configure CUDA execution provider:
```python
import onnxruntime as ort

session = ort.InferenceSession(
    "model.onnx",
    providers=['CUDAExecutionProvider', 'CPUExecutionProvider']
)

# Or with explicit options:
providers = [("CUDAExecutionProvider", {"device_id": 0})]
sess = ort.InferenceSession("model.onnx", providers=providers)
```

## LiveKit Agent Integration

LiveKit Agents framework expects an STT plugin passed to AgentSession:
```python
from livekit.agents import AgentSession
from livekit.plugins import silero

session = AgentSession(
    stt=CustomSTTPlugin(),  # Voxtral ONNX wrapper
    vad=silero.VAD.load(),
    # ... llm, tts
)
```

Custom STT plugins must implement the LiveKit STT interface for streaming recognition.

## Docker Considerations

For GPU access in Docker:
- Use NVIDIA runtime: `--gpus all` or specific GPU IDs
- Base image: `nvidia/cuda:12.x-cudnn-runtime-ubuntu22.04`
- Install CUDA-compatible ONNX runtime inside container

## Benchmark Metrics to Track

- First-packet latency (ms)
- Real-time factor (processing time / audio duration)
- WER (Word Error Rate) for zh-TW/English mixed speech
- GPU memory utilization
- Throughput (concurrent streams)

## Current Date

2026-04-15

---
> Source: [vieenrose/realtime-stt](https://github.com/vieenrose/realtime-stt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
