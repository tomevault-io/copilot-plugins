## tts-stt-api

> - **Endpoint:** `POST /v1/audio/speech`

# TTS & STT API Endpoints

## Text-to-Speech (TTS)
- **Endpoint:** `POST /v1/audio/speech`
- **Authentication:** `Authorization: Bearer <API_KEY>`
- **Request Body:** JSON, see [src/models.py](mdc:src/models.py) `TTSRequest`
  - `model` (string): TTS model name (e.g., "tts-1")
  - `input` (string): Text to synthesize
  - `voice` (string): Voice type (e.g., "alloy")
  - `response_format` (string, optional): Output format (e.g., "mp3")
  - `speed` (float, optional): Speech speed
- **Response:** Audio stream (e.g., mp3)
- **Implementation:** [src/main.py](mdc:src/main.py), [src/tts_logic.py](mdc:src/tts_logic.py)

## Speech-to-Text (STT)
- **Endpoint:** `POST /v1/audio/transcriptions`
- **Authentication:** `Authorization: Bearer <API_KEY>`
- **Request Body:** Multipart form
  - `file`: Audio file (mp3, wav, etc.)
  - `model` (string, optional): STT model name
  - `language` (string, optional): Language code
  - `prompt` (string, optional): Prompt for style/continuation
  - `response_format` (string, optional): Output format (default: json)
  - `temperature` (float, optional): Sampling temperature
- **Response:** JSON with transcription text
- **Implementation:** [src/main.py](mdc:src/main.py), [src/stt_logic.py](mdc:src/stt_logic.py)

## Models
- See [src/models.py](mdc:src/models.py) for Pydantic request/response models.

---
> Source: [mobailabs/mac-dia-server](https://github.com/mobailabs/mac-dia-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
