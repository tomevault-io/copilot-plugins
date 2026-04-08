## simple-llm-service

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a REST API service that provides OpenAI-compatible endpoints for serving Large Language Models (LLMs). It supports both llama.cpp (GGUF format) and MLX backends for model inference.

## Common Development Commands

### Setup
```bash
# Initial setup (creates .venv, installs dependencies)
./setup.sh

# Activate virtual environment
source .venv/bin/activate
```

### Running the Service
```bash
# Start the service (recommended - handles venv activation)
./start_service.sh

# Start with development reload (activate venv first)
uvicorn server:app --reload

# Start with custom host/port
PORT=8080 HOST=127.0.0.1 python server.py
```

### Testing
```bash
# Run API tests
python test_api.py

# Test specific endpoint manually
curl http://localhost:8000/v1/models
```

### Configuration
```bash
# Copy and edit configuration
cp .env.example .env
```

## Architecture

### Core Components

1. **server.py**: FastAPI application implementing OpenAI-compatible endpoints
   - `/v1/chat/completions` - Chat completions with streaming support
   - `/v1/completions` - Text completions (legacy format)
   - `/v1/embeddings` - Generate embeddings with BGE-M3 dense/sparse support
   - `/v1/rerank` - Rerank documents using cross-encoder models
   - `/v1/models` - List available models

2. **model_manager.py**: Handles model loading and inference
   - Supports both llama.cpp and MLX backends
   - Five-tier model system (LIGHT, MEDIUM, HEAVY, EMBEDDING, RERANKER)
   - Lazy loading: models are loaded on first use, not at startup
   - Async queue processing per tier

3. **embedders.py**: Modular embedding implementations
   - Base embedder interface for easy extension
   - BGE-M3 embedder with dense/sparse/ColBERT support
   - Simple embedder for general models

4. **rerankers.py**: Modular reranker implementations
   - Base reranker interface for easy extension
   - BGE-reranker for cross-encoder models
   - Embedding-based reranker for similarity scoring

5. **cache.py**: Response caching with optional disk persistence
   - LRU eviction strategy
   - Cache keys based on request parameters
   - Separate caching for embeddings

### Key Design Patterns

- **OpenAI API Compatibility**: Drop-in replacement for OpenAI API clients
- **Async Architecture**: Non-blocking request handling using FastAPI
- **Multi-Backend Support**: Abstraction layer for different inference engines
- **Environment-Based Configuration**: All settings via .env file

### Configuration Structure

The service uses a five-tier model system configured in .env:
- `LIGHT_MODEL_*`: For fast, simple tasks
- `MEDIUM_MODEL_*`: For balanced performance
- `HEAVY_MODEL_*`: For complex reasoning tasks
- `EMBEDDING_MODEL_*`: For embedding models (BGE-M3, etc.)
- `RERANKER_MODEL_*`: For reranking models (BGE-reranker, etc.)

Each tier has independent settings for:
- Model path
- Context length
- Temperature and sampling parameters
- Number of processing threads
- Embedding mode and dimension (for EMBEDDING tier)

### Embedding Model Configuration

To use BGE-M3 or other embedding models:
```bash
EMBEDDING_MODEL_PATH=/path/to/bge-m3-Q8_0.gguf
EMBEDDING_BACKEND=llamacpp
EMBEDDING_EMBEDDING_MODE=true
EMBEDDING_EMBEDDING_DIMENSION=1024
EMBEDDING_N_CTX=8192
```

### BGE-M3 Features

The service supports BGE-M3's advanced embedding capabilities:
- **Dense embeddings**: Standard 1024-dimensional embeddings
- **Sparse embeddings**: Token-based sparse representations for hybrid search
- **ColBERT embeddings**: Multi-vector representations (future support)

Use the `/v1/embeddings` endpoint with:
```json
{
  "model": "embedding",
  "input": "Your text here",
  "embedding_type": "dense",  // or "sparse", "colbert"
  "return_sparse": true       // Return both dense and sparse
}
```

### Reranker Configuration

To use BGE-reranker or other cross-encoder models:
```bash
RERANKER_MODEL_PATH=/path/to/bge-reranker-v2-m3-Q8_0.gguf
RERANKER_BACKEND=llamacpp
RERANKER_N_CTX=512
RERANKER_TEMPERATURE=0.01
```

## Development Notes

- No formal test framework; uses simple HTTP-based testing via test_api.py
- No linting configuration; follow existing code style
- Type hints used throughout via Pydantic models
- Logging configured via Python's logging module
- CORS enabled by default for browser-based applications

## Structured Output Implementation

### MLX Backend with Outlines
The structured output feature for MLX models uses the Outlines library for constrained JSON generation:
- Integrated via `outlines.models.from_mlxlm()` to wrap MLX models
- Uses `outlines.generate.json()` with flexible schema for JSON object generation
- Ensures valid JSON output every time (no more parsing errors)
- Falls back to prompt-based generation if Outlines fails

### Important Notes
1. **Model Path Configuration**: Ensure MEDIUM_MODEL_PATH points to the actual MLX model directory (e.g., `/path/to/mlx-community/gemma-3-4b-it-qat-4bit`), not just metadata
2. **Clear Prompts Required**: When using `response_format: {"type": "json_object"}`, be explicit about the JSON structure you want
3. **Empty JSON on Vague Prompts**: The model may return `{}` if the prompt doesn't clearly specify what fields to include

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/malibrated)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/malibrated)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
