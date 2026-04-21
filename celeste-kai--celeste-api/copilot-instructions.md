## celeste-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`celeste-api` is the FastAPI backend service for the Celeste multi-modal AI framework. It provides unified HTTP endpoints for all Celeste capabilities including text generation, image generation/editing, video generation, audio transcription, text-to-speech, document intelligence, and reranking.

## Architecture

### Core Components
- **FastAPI Application**: Main application in `src/celeste_api/main.py`
- **Route Modules**: Organized by capability in `src/celeste_api/routes/`
- **CORS Middleware**: Configurable cross-origin support for frontend integration
- **Dynamic Client Creation**: Uses factory functions from celeste packages

### Route Structure
Each route module follows a consistent pattern:
- Located in `src/celeste_api/routes/{capability}.py`
- Uses APIRouter with `/v1` prefix and appropriate tags
- Creates clients dynamically using celeste package factory functions
- Returns standardized JSON responses

### Current Routes
- `/v1/text/*` - Text generation (sync and streaming)
- `/v1/images/*` - Image generation and editing
- `/v1/videos/*` - Video generation
- `/v1/audio/*` - Audio generation (TTS) and proxy
- `/v1/documents/*` - Document analysis and intelligence
- `/v1/rerank` - Text reranking
- `/v1/discovery/*` - Capabilities, providers, and models discovery

## Development Commands

### Setup
```bash
uv sync                           # Install dependencies
uv add <package>                  # Add new dependency (never use pip install)
```

### Running the API
```bash
uvicorn celeste_api.main:app --reload --port 8000  # Development server
make api                          # Run from parent directory
```

### Code Quality
```bash
ruff check src/                   # Linting
ruff format src/                  # Formatting
mypy src/                        # Type checking
pre-commit run --all-files        # Run all pre-commit hooks
```

### Testing
```bash
# Manual testing with curl
curl http://localhost:8000/v1/health
curl http://localhost:8000/v1/capabilities

# Interactive docs
# Visit http://localhost:8000/docs for Swagger UI
# Visit http://localhost:8000/redoc for ReDoc
```

## Adding New Routes

### Steps to Add a New Capability Route
1. Create route file in `src/celeste_api/routes/{capability}.py`
2. Define APIRouter with appropriate prefix and tags
3. Implement endpoint functions using celeste packages
4. Import and include router in `main.py`
5. Export from `routes/__init__.py`

### Route Implementation Pattern
```python
from __future__ import annotations

from celeste_{capability} import create_{capability}_client
from fastapi import APIRouter

router = APIRouter(prefix="/v1", tags=["{capability}"])

@router.post("/{capability}/action")
async def perform_action(payload: dict) -> dict:
    provider = payload["provider"]
    model = payload.get("model")
    # ... extract other parameters

    client = create_{capability}_client(provider, model=model)
    result = await client.method(...)

    return {
        "result": result,
        "provider": provider,
        "model": model,
        "metadata": {...}
    }
```

### File Upload Pattern
For endpoints requiring file uploads:
```python
from fastapi import File, UploadFile

@router.post("/endpoint")
async def process_file(
    file: UploadFile = File(...),
    provider: str = Form(...),
    model: str = Form(None)
) -> dict:
    content = await file.read()
    # Process file content
```

## Environment Configuration

### Required Environment Variables
API keys are managed through celeste-core settings:
- `GOOGLE_API_KEY` - Google services
- `OPENAI_API_KEY` - OpenAI services
- `ANTHROPIC_API_KEY` - Anthropic services
- `MISTRAL_API_KEY` - Mistral services
- `HUGGINGFACE_TOKEN` - HuggingFace services
- `REPLICATE_API_TOKEN` - Replicate services
- `COHERE_API_KEY` - Cohere services
- `TOPAZ_LABS_API_KEY` - Topaz Labs services

### CORS Configuration
- `CORS_ALLOW_ORIGINS` - Comma-separated list of allowed origins (default: "*")

## Response Patterns

### Standard Response Format
```json
{
  "content": "...",           // Main response content
  "provider": "provider_id",  // Provider used
  "model": "model_id",        // Model used
  "metadata": {               // Additional information
    "usage": {...},
    "latency": 0.123,
    ...
  }
}
```

### Streaming Response
Uses NDJSON (newline-delimited JSON) format:
```json
{"content": "chunk1", "metadata": {"is_stream_chunk": true}}
{"content": "chunk2", "metadata": {"is_stream_chunk": true}}
```

## Dependencies

### Core Dependencies
- `fastapi>=0.111.0` - Web framework
- `uvicorn[standard]>=0.30.0` - ASGI server
- `pydantic>=2.8.0` - Data validation
- `python-multipart>=0.0.9` - Multipart form support

### Celeste Packages
All celeste packages are referenced via git sources:
- `celeste-core` - Base classes and enums
- `celeste-client` - Text generation
- `celeste-image-generation` - Image generation
- `celeste-image-edit` - Image editing
- `celeste-video-generation` - Video generation
- `celeste-reranking` - Text reranking
- `celeste-document-intelligence` - Document processing

## Important Notes

- Always use `uv add` instead of `pip install` for dependencies
- Follow async/await patterns for all endpoint handlers
- Use provider factory functions from celeste packages
- Handle errors gracefully with appropriate HTTP status codes
- Include proper type hints for better IDE support
- CORS is configured for development; adjust for production

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/celeste-kai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
