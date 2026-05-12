## voxtral-wyoming

> Offline Speech-to-Text service using Mistral Voxtral models with [Wyoming protocol](https://github.com/OHF-Voice/wyoming) for Home Assistant integration.

# AGENTS.md

Offline Speech-to-Text service using Mistral Voxtral models with [Wyoming protocol](https://github.com/OHF-Voice/wyoming) for Home Assistant integration.

## Tech Stack

- Python 3.13, managed with `uv`
- Key deps: `transformers`, `torch`, `wyoming`, `mistral-common[audio]`
- Build: setuptools, entry point `voxtral-wyoming` -> `voxtral_wyoming.server:cli`

## Project Structure

```
src/voxtral_wyoming/
  __init__.py          # Package version
  server.py            # CLI entry point, async Wyoming TCP server
  audio.py             # Audio utilities (clamping, WAV saving, PCM helpers)
  transcriber/
    __init__.py
    base.py            # ITranscriber protocol + TranscriptionResult dataclass
    voxtral.py         # VoxtralTranscriber impl (gen1 + gen2 auto-detection)
```

- `Dockerfile`, `docker-compose.yml`, `docker-compose.gpu.yml` for containerized deployment
- `.env.example` documents all environment variables with defaults
- `examples/client_sample.py` for testing

## Architecture

- **server.py**: Async TCP server implementing Wyoming ASR protocol (Describe/Transcribe/AudioStart/AudioChunk/AudioStop events). Config loaded from env vars via `python-dotenv`.
- **transcriber/base.py**: `ITranscriber` Protocol — accepts PCM16 mono bytes, returns `TranscriptionResult`.
- **transcriber/voxtral.py**: Loads Voxtral model via HuggingFace `transformers` (_not_ via the alternative vLLM). Auto-detects model generation at load time (gen2 `VoxtralRealtimeForConditionalGeneration` vs gen1 `VoxtralForConditionalGeneration`). Supports two transcription modes: transcribe-only (default) and chat mode with system prompt.
- Device auto-detection: CUDA > MPS > CPU, with CPU fallback on failure.
- dtype auto-detected from model files unless overridden via `DATA_TYPE` env var.

## Dev Setup

```bash
uv venv && source .venv/bin/activate && uv sync
voxtral-wyoming              # uses .env
voxtral-wyoming path/to.env  # custom env file
```

## Key Conventions

- All config via environment variables (no CLI flags beyond optional env file path)
- Audio format: PCM16 mono, little-endian, default 16kHz
- Model is eagerly loaded at startup to avoid slow first request
- No tests exist yet in the repository, but feel free to add tests

## External Documentation
- [Voxtral-Mini-4B-Realtime-2602 model page on Hugging Face](https://huggingface.co/mistralai/Voxtral-Mini-4B-Realtime-2602)
- [Voxtral Realtime Transformers](https://huggingface.co/docs/transformers/main/en/model_doc/voxtral_realtime)
- [Voxtral-Mini-3B-2507 model page on Hugging Face](https://huggingface.co/mistralai/Voxtral-Mini-3B-2507)

---
> Source: [Johnson145/voxtral_wyoming](https://github.com/Johnson145/voxtral_wyoming) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
