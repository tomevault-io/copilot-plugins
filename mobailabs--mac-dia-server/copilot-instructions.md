## project-structure

> This project is a FastAPI server providing OpenAI-compatible Text-to-Speech (TTS) and Speech-to-Text (STT) API endpoints using `mlx-audio` for Apple Silicon.

# Project Structure Guide

This project is a FastAPI server providing OpenAI-compatible Text-to-Speech (TTS) and Speech-to-Text (STT) API endpoints using `mlx-audio` for Apple Silicon.

## Main Entry Points
- [start.py](mdc:start.py): Launches the server by calling `start_server()` from [src/main.py](mdc:src/main.py).
- [src/main.py](mdc:src/main.py): Defines the FastAPI app, API endpoints, and server startup logic.

## API Endpoints
- **POST** `/v1/audio/speech`: Text-to-Speech (TTS) endpoint. See [src/main.py](mdc:src/main.py) and [src/tts_logic.py](mdc:src/tts_logic.py).
- **POST** `/v1/audio/transcriptions`: Speech-to-Text (STT) endpoint. See [src/main.py](mdc:src/main.py) and [src/stt_logic.py](mdc:src/stt_logic.py).

## Core Logic
- [src/tts_logic.py](mdc:src/tts_logic.py): Implements TTS generation using `mlx-audio`.
- [src/stt_logic.py](mdc:src/stt_logic.py): Implements STT (transcription) using `mlx-audio` Whisper model.
- [src/models.py](mdc:src/models.py): Pydantic models for request/response validation.
- [src/security.py](mdc:src/security.py): API key authentication logic.

## Configuration
- [README.md](mdc:README.md): Project overview, setup, and API usage instructions.
- [pyproject.toml](mdc:pyproject.toml): Project metadata and dependencies.
- `.env`: Stores the API key for authentication (not in version control).

## Usage
- Run the server: `uv run start.py`
- See [README.md](mdc:README.md) for detailed API usage and curl examples.

---
> Source: [mobailabs/mac-dia-server](https://github.com/mobailabs/mac-dia-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
