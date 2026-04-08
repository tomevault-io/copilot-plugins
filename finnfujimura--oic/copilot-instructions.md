## oic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Offline voice field assistant for Raspberry Pi 5 (16GB RAM). Core loop: **push-to-talk → WAV → STT → LLM → TTS → playback**. Currently testing on Windows 11 before Pi deployment.

## Architecture

Python orchestrates three CLI-based inference engines via subprocess:
- **STT:** `whisper.cpp` (base.en model) → `bin/whisper/Release/whisper-cli.exe`
- **LLM:** `llama.cpp` (2B-3B GGUF Q4_K_M) → `bin/llama/llama-cli.exe`
- **TTS:** `piper` → `bin/piper/piper/piper.exe`

Python handles coordination only; all heavy compute runs in native binaries.

## Key Files

- `app/main.py` - Orchestrator with push-to-talk loop (SPACE key)
- `app/config.py` - All paths, settings, env var overrides
- `app/state.py` - ConversationState class manages history + context

## Backend Wrapper Contracts

Each wrapper returns a Result dataclass with `success: bool`, `time_ms: float`, and component-specific outputs:

```python
# stt/whispercpp.py
STTResult(success, transcript, time_ms)

# llm/llamacpp.py
LLMResult(success, response, time_ms, tokens_per_sec)

# tts/piper.py
TTSResult(success, output_path, time_ms)
```

## Development Commands

```bash
# Run the voice assistant
python app/main.py

# List audio devices
python app/main.py --list-devices

# Enable debug logging
python app/main.py --debug

# Debug: test imports and keyboard detection
python debug_imports.py

# Debug: step-by-step component testing
python debug_pipeline.py
```

## Environment Overrides

All paths in `config.py` can be overridden via environment variables:
- `WHISPER_CPP_PATH`, `LLAMA_CPP_PATH`, `PIPER_PATH` - binary paths
- `WHISPER_MODEL`, `LLM_MODEL`, `PIPER_MODEL`, `PIPER_CONFIG` - model paths
- `LOG_LEVEL` - logging verbosity (DEBUG/INFO/WARN/ERROR)

## Key Constraints

- **Fully offline** - no network dependencies at runtime
- **No ML frameworks** - subprocess to CLI tools only
- **Audio format:** mono, 16 kHz, 16-bit PCM WAV
- **LLM defaults:** temp=0.3, max_tokens=180, context=2048, threads=4
- **Model paths:** `/mnt/nvme/models/` with fallback to `./models/`

## Platform Notes

- **Windows:** Uses `keyboard` library for key detection
- **Linux/Pi:** Uses `pynput` (or GPIO button on Pi)
- **Pi deployment:** NVMe SSD preferred, 5V/5A power required

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/finnfujimura)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/finnfujimura)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
