## local-livekit-plugins

> LiveKit Local Plugins - Custom STT and TTS plugins for LiveKit Agents that run entirely locally without cloud APIs.

# CLAUDE.md

## Project Overview

LiveKit Local Plugins - Custom STT and TTS plugins for LiveKit Agents that run entirely locally without cloud APIs.

## Architecture

- `src/local_livekit_plugins/` - The installable Python package
  - `faster_whisper_stt.py` - STT plugin using faster-whisper (GPU-accelerated)
  - `piper_tts.py` - TTS plugin using Piper
- `examples/` - Demo code showing how to use the plugins (not part of installed package)

## Key Technical Notes

- **Plugin imports must be at module level** - LiveKit requires plugins to be registered on the main thread. Never import plugins inside functions.
- **PyTorch provides CUDA libraries** - torch dependency bundles nvidia-cublas-cu12 etc. for faster-whisper GPU support
- **Piper CUDA has version constraints** - CPU mode (`use_cuda=False`) is recommended due to CUDA 12+ compatibility issues

## Development Commands

```bash
uv sync                              # Install dependencies
uv run examples/voice_agent.py dev   # Run with LiveKit playground
uv run examples/voice_agent.py console  # Run in console mode
uv run ruff check src/               # Lint
uv run mypy src/                     # Type check
uv run pytest                        # Test
```

## Environment

Requires `.env.local` in examples/ directory (copy from `.env.local.example`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CoreWorxLab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
