## ha-voice-rocm

> Wyoming protocol speech services (STT + TTS) for Home Assistant, targeting AMD GPUs via ROCm 7.1.1. Docker Compose orchestrates all services. Target hardware: Ryzen AI Max 395+ (Radeon 8060S, gfx1151/RDNA 3.5).

# CLAUDE.md

## Project Overview

Wyoming protocol speech services (STT + TTS) for Home Assistant, targeting AMD GPUs via ROCm 7.1.1. Docker Compose orchestrates all services. Target hardware: Ryzen AI Max 395+ (Radeon 8060S, gfx1151/RDNA 3.5).

## Repository Structure

```
├── docker-compose.yml          # All services defined here
├── .env.example                # All config vars with docs
├── <service>/
│   ├── Dockerfile              # ROCm base image + deps
│   ├── entrypoint.sh           # Container entrypoint
│   ├── <name>_wrapper.py       # Wyoming AsyncServer (main)
│   └── <name>_handler.py       # Wyoming EventHandler (logic)
```

### Services

| Service | Type | Port | GPU | Notes |
|---------|------|------|-----|-------|
| whisper | STT | 10300 | Yes (CTranslate2) | ROCm fork of ctranslate2 |
| moonshine | STT | 10302 | No (ONNX/CPU) | Ultra-low latency |
| voxtral | STT | 10301 | Yes (vLLM) | Not working yet |
| qwen3-tts | TTS | 10200 | Yes (PyTorch) | Voice design via text |
| chatterbox-turbo | TTS | 10201 | Yes (PyTorch) | Needs HF_TOKEN |
| pocket-tts | TTS | 10202 | No (CPU) | Ultra-low latency |
| kokoro | TTS | 10203 | No (API proxy) | Proxies to Kokoro-FastAPI |
| parakeet | STT | 10303 | Yes (PyTorch) | NeMo parakeet-tdt-0.6b-v3 |
| piper | TTS | — | — | Deprecated, commented out |

## Code Patterns

- Each service is a standalone Wyoming protocol server using `wyoming` Python package
- `*_wrapper.py` sets up `AsyncServer` with `Info` metadata and CLI arg parsing
- `*_handler.py` implements `AsyncEventHandler` with `handle_event()` for `Synthesize`/`Transcribe` events
- Audio flows as Wyoming `AudioChunk` events over TCP
- GPU services use `rocm/pytorch:rocm7.1.1_ubuntu24.04_py3.12_pytorch_release_2.9.1` base image
- CPU-only services use lighter base images

## Development

- Config: copy `.env.example` → `.env`, set `HSA_OVERRIDE_GFX_VERSION` for your GPU
- Build: `docker compose build <service-name>`
- Run: `docker compose up -d <service-name>`
- Logs: `docker compose logs -f <service-name>`
- The compose file runs services on a remote Docker host — images build and run there, not locally
- Model weights are persisted in `./data`, `./<service>-data` volume mounts
- No tests, no CI, no linting configured — this is an experimental/hobbyist project

---
> Source: [ndom91/ha-voice-rocm](https://github.com/ndom91/ha-voice-rocm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
