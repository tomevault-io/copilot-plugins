## sesame-csm-openai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python FastAPI application that provides an OpenAI-compatible Text-to-Speech (TTS) API using two different models:
- **CSM-1B**: Sesame's Conversational Speech Model (default)
- **Dia-1.6B**: Nari Labs' Dialogue Speech Model

The application runs in Docker containers with CUDA support for GPU acceleration.

## Development Commands

### Building and Running
```bash
# Build and start with Docker Compose
docker compose up -d --build

# Build with specific TTS engine
TTS_ENGINE=csm docker compose up -d --build
TTS_ENGINE=dia docker compose up -d --build

# Run locally for development
python -m app.main

# Run with development mode (auto-reload)
DEV_MODE=true python -m app.main
```

### Environment Variables
Key environment variables for configuration:
- `TTS_ENGINE`: Choose between "csm" (default) or "dia"
- `HF_TOKEN`: Hugging Face token for model access
- `CSM_DEVICE_MAP`: Multi-GPU mapping ("auto", "balanced", "sequential")
- `DEV_MODE`: Enable development mode with auto-reload
- `LOG_LEVEL`: Set logging level (INFO, DEBUG, WARNING, ERROR)

### Testing

The project now includes a comprehensive test suite using pytest. Run tests with:

```bash
# Run all tests
pytest

# Run with coverage report
pytest --cov=app --cov-report=html --cov-report=term

# Run only unit tests
pytest -m unit

# Run only integration tests
pytest -m integration

# Run tests in parallel (faster)
pytest -n auto

# Run with verbose output
pytest -v

# Run a specific test file
pytest tests/unit/test_schemas.py

# Run tests with specific markers
pytest -m "not gpu"  # Skip GPU tests
pytest -m "not slow"  # Skip slow tests
```

#### Test Structure
- **Unit Tests** (`tests/unit/`): Test individual components in isolation
  - `test_schemas.py`: API request/response validation
  - `test_text_normalizer.py`: Text processing and normalization
  - `test_prompt_engineering.py`: Voice formatting and segmentation
  - `test_audio_utils.py`: Audio format conversion
  - `test_voice_cloning.py`: Voice cloning system (mocked)
  - `test_generator.py`: TTS generator (mocked)
  
- **Integration Tests** (`tests/integration/`):
  - `test_api_routes.py`: Full API endpoint testing with mocked backends

- **Test Configuration**:
  - `pytest.ini`: Pytest configuration and markers
  - `conftest.py`: Shared fixtures and test setup

#### Manual Testing
For manual testing of the live API:
```bash
# Health check
curl http://localhost:8000/health

# Generate speech
curl -X POST http://localhost:8000/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{"model": "csm-1b", "input": "Hello world", "voice": "alloy"}' \
  --output test.mp3
```

## Architecture

### Core Components

**FastAPI Application** (`app/main.py`):
- Main application entry point with lifespan management
- Handles model loading, voice systems initialization
- Configures CORS, static files, and routing

**TTS Generator** (`app/generator.py`):
- Core speech generation logic for CSM-1B model
- Handles text tokenization, audio generation, and watermarking
- Supports multi-GPU distribution and optimization

**API Routes** (`app/api/routes.py`):
- OpenAI-compatible speech generation endpoints
- Handles voice mapping, format conversion, and streaming
- Supports both standard and cloned voices

**Voice Systems**:
- **Voice Cloning** (`app/voice_cloning.py`): Clone voices from audio samples
- **Voice Enhancement** (`app/voice_enhancement.py`): Improve voice consistency
- **Voice Memory** (`app/voice_memory.py`): Maintain voice context across requests

**Model Adapters**:
- **CSM-1B**: Native support via torchtune models
- **Dia-1.6B**: Adapter pattern in `app/dia_adapter.py`

### Key Features

1. **OpenAI API Compatibility**: Drop-in replacement for OpenAI TTS API
2. **Voice Cloning**: Clone voices from audio samples with web UI
3. **Streaming Support**: Real-time audio streaming
4. **Multi-GPU Support**: Distributed inference across multiple GPUs
5. **Voice Consistency**: Maintains voice characteristics across requests
6. **YouTube Voice Cloning**: Extract and clone voices from YouTube videos

### Data Flow

1. **Request Processing**: FastAPI receives TTS request with text and voice
2. **Voice Resolution**: Maps voice name/ID to speaker ID or cloned voice
3. **Text Processing**: Normalizes text and applies prompt engineering
4. **Audio Generation**: 
   - CSM-1B: Uses transformer-based generation with audio tokenization
   - Dia-1.6B: Uses adapted interface with S1/S2 speaker tags
5. **Post-Processing**: Applies voice enhancement, speed adjustment, format conversion
6. **Response**: Returns audio in requested format (MP3, WAV, etc.)

## Important File Locations

- **Model Storage**: `/app/models/` (persistent volume)
- **Cloned Voices**: `/app/cloned_voices/` (persistent volume)
- **Voice References**: `/app/voice_references/` (persistent volume)
- **Audio Cache**: `/app/audio_cache/` (persistent volume)
- **Logs**: `/app/logs/` 
- **Static Files**: `/app/static/` (web UI)

## Performance Optimizations

The application includes several performance optimizations:
- **Mixed Precision**: Uses bfloat16/float16 when available
- **Torch Compile**: JIT compilation for faster inference
- **CUDA Optimizations**: TF32, Flash Attention, CUDNN benchmark
- **Audio Caching**: Caches generated audio by content hash
- **Batch Generation**: Generates multiple audio frames at once

## API Endpoints

### Core TTS Endpoints
- `POST /v1/audio/speech` - Generate speech (OpenAI compatible)
- `POST /v1/audio/speech/streaming` - Stream audio generation
- `GET /v1/audio/voices` - List available voices
- `GET /v1/audio/models` - List available models

### Voice Cloning Endpoints
- `POST /v1/voice-cloning/clone` - Clone voice from audio
- `GET /v1/voice-cloning/voices` - List cloned voices
- `POST /v1/voice-cloning/generate` - Generate with cloned voice
- `DELETE /v1/voice-cloning/voices/{voice_id}` - Delete cloned voice

### Utility Endpoints
- `GET /health` - Health check with system status
- `GET /voice-cloning` - Voice cloning web UI
- `GET /debug` - Debug information
- `GET /version` - API version information

## Voice System Architecture

The application uses a sophisticated voice system with three main components:

1. **Standard Voices**: Pre-configured voices (alloy, echo, fable, onyx, nova, shimmer)
2. **Voice Cloning**: Create custom voices from audio samples
3. **Voice Enhancement**: Improve consistency and quality

Voice data is persisted across container restarts using Docker volumes.

## Development Notes

- **Comprehensive Test Suite**: Full pytest-based test coverage with unit and integration tests
- **Docker-First**: Designed to run in containerized environment
- **GPU Required**: Optimal performance requires CUDA-capable GPU
- **Model Downloads**: Models are downloaded at build time or runtime
- **Persistent Storage**: Important data stored in Docker volumes for persistence

## Testing Best Practices

When adding new features or fixing bugs:

1. **Write Tests First**: Follow TDD principles when possible
2. **Mock Heavy Dependencies**: Mock GPU operations, model loading, and external APIs
3. **Test Edge Cases**: Include tests for error conditions and boundary values
4. **Maintain Coverage**: Aim for >80% code coverage on new code
5. **Use Fixtures**: Leverage pytest fixtures in `conftest.py` for common test needs
6. **Mark Tests Appropriately**: Use markers (unit, integration, gpu, slow) to categorize tests

---
> Source: [phildougherty/sesame_csm_openai](https://github.com/phildougherty/sesame_csm_openai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
