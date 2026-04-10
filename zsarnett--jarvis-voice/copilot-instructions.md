## jarvis-voice

> Jarvis is a voice FRONTEND to the hub agent system. It is NOT a standalone assistant.

# Jarvis Voice Interface

## Project Overview
Jarvis is a voice FRONTEND to the hub agent system. It is NOT a standalone assistant.
Same pattern as Discord: voice in -> STT -> text -> hub processes -> text back -> TTS -> voice out.

The ClaudeBridge runs `claude -p` from the workspace root (`~/Documents/ZacksWorkspace/`)
so it picks up the hub's CLAUDE.md and has access to all the same tools, dispatch scripts,
SecondBrain, MCP integrations, etc.

## Architecture: Hub Routing

Jarvis is another input channel to the hub, just like Discord:
- Discord: text in -> hub processes -> text out (to Discord channel)
- Jarvis voice: voice in -> STT -> text -> hub processes -> text back -> TTS -> voice out
- Jarvis web: text/voice in -> hub processes -> text back -> display + optional TTS

The `ClaudeBridge` spawns `claude -p` with `cwd=workspace_root` and full tool access.
This is functionally equivalent to the hub agent -- same CLAUDE.md, same tools, same capabilities.

## Conventions

### Python
- Python 3.11+, use venv (`.venv/`)
- Type hints on all function signatures
- asyncio for concurrency
- No emoji in code, comments, or output

### Audio Pipeline
- Sample rate: 16kHz, 16-bit, mono throughout
- All audio as numpy arrays (float32 normalized) or bytes (int16 PCM)
- Thread-safe queues for audio data between components

### Components
- Each component is a separate module in `jarvis/`
- Components communicate via typed events (defined in `events.py`)
- Configuration via dataclass in `config.py`, loaded from `~/.jarvis/config.yaml`
- LLM interface is behind `ClaudeBridge` abstraction (future Agent SDK migration)

### Testing
- Tests in `jarvis/tests/`
- `pytest` for unit tests
- Mock `claude -p` subprocess for bridge tests

### Key Files
- `jarvis/main.py` -- CLI entry point (push-to-talk / always-on voice)
- `jarvis/claude_bridge.py` -- hub-routed LLM integration via `claude -p`
- `jarvis/state_machine.py` -- always-on voice loop (wake word -> listen -> process -> speak)
- `jarvis/sentence_buffer.py` -- streaming text to sentence chunking for TTS
- `jarvis/tts.py` -- Kokoro ONNX TTS wrapper
- `jarvis/stt.py` -- openai-whisper STT wrapper
- `jarvis/audio_engine.py` -- sounddevice I/O with AEC
- `web/server.py` -- FastAPI web interface (voice + text chat + dashboard)
- `web/static/index.html` -- SPA frontend

### Running
```bash
# Voice mode (local mic)
source .venv/bin/activate
python -m jarvis --mode voice

# Web interface
source .venv/bin/activate
python web/server.py
# Then open http://localhost:3000
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zsarnett)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zsarnett)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
