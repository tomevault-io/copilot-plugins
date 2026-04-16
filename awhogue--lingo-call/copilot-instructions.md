## lingo-call

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Run Commands

```bash
# Install (requires Python 3.10-3.12, recommend 3.11)
pip install --upgrade pip setuptools wheel numpy
pip install --no-build-isolation chatterbox-tts
pip install -e ".[llm,stt]"

# Run in text mode (type input, no STT)
python cli.py --text-mode

# Run in voice mode
python cli.py

# Run with different model
python cli.py --llm-model Qwen/Qwen2.5-3B-Instruct

# Test components
python cli.py --test

# Lint/format
ruff check src/
black src/
mypy src/
```

## Architecture

The application uses a pipeline architecture with swappable provider implementations:

**Pipeline Flow:** Audio → STT → LLM → TTS → Audio

- `ConversationPipeline` (pipeline.py) orchestrates all components
- `ConversationManager` (conversation.py) tracks message history
- `AppConfig` (config.py) contains nested dataclass configs for each component

**Provider Pattern:** Each ML component has an abstract base class in `{component}/base.py` with a concrete implementation:

| Component | Base Class | Implementation | Library |
|-----------|------------|----------------|---------|
| STT | `STTProvider` | `OmnilingualSTT` | omnilingual-asr |
| LLM | `LLMProvider` | `TransformersLLM` | HuggingFace transformers |
| TTS | `TTSProvider` | `ChatterboxTTS` | chatterbox-tts |

**Language Handling:** The `SUPPORTED_LANGUAGES` dict in config.py maps short codes (e.g., "es") to tuples of (stt_lang, tts_lang, display_name). Each provider uses its specific format.

## Key Implementation Notes

- LLM loads eagerly at startup (not on first use) - see `TransformersLLM(lazy=False)` in pipeline.py
- Chatterbox requires patching `torch.load` with `map_location` for non-CUDA machines (see chatterbox.py)
- TTS sample rate comes from the model at runtime, not from config
- Text mode (`--text-mode`) skips STT/recorder initialization entirely
- 4-bit quantization is enabled by default for LLM; disable with `--no-4bit`

## Dependency Constraints

- Python must be <3.13 due to chatterbox-tts requiring numpy<1.26
- chatterbox-tts must be installed with `--no-build-isolation` due to build dependencies
- There's a torch version conflict between chatterbox-tts (2.6.0) and fairseq2 (2.8.0); use `--no-deps` if needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/awhogue) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
