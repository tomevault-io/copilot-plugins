## homebrew-uttr

> python development project


# Python Development Guidelines

## 🎯 Python Component Responsibilities
- **ML Inference**: Whisper STT processing
- **LLM Processing**: Text cleanup and formatting
- **Configuration**: Settings management
- **CLI Interface**: Command-line scripts for Swift integration

## 📋 Python File Conventions

### transcribe.py (Main Entry Point)
- Accept audio file path as command line argument
- Orchestrate Whisper STT → LLM processing pipeline
- Output final text to stdout for Swift consumption
- Handle errors gracefully with proper exit codes

### whisper_wrapper.py
- Integrate with Whisper (base or faster models)
- Handle audio preprocessing (format, sample rate)
- Optimize for speed vs accuracy trade-offs

### llm_processor.py
- Integrate with Ollama for local LLM
- Create a configurable prompt for processing

## 🔧 Python Best Practices
- Use type hints for all function signatures
- Implement proper logging with `logging` module
- Use `argparse` for CLI argument parsing
- Handle audio files with `soundfile`
- Use `asyncio` for async operations if needed
- Implement proper error handling and exit codes
- Use uv for package management 

## 📦 Key Dependencies
- `whisper` for STT
- `ollama` for LLM
- `sounddevice` for audio processing
- `numpy` for numerical operations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rakk301) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
