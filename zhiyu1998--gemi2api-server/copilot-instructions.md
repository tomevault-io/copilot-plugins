## gemi2api-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

This is a FastAPI-based server that provides OpenAI-compatible API endpoints for Google's Gemini AI model via the `gemini-webapi` library. The server acts as a bridge, translating OpenAI API requests to Gemini API calls.

### Core Components

- **main.py**: Single-file application containing all API endpoints, authentication, and request handling
- **Authentication**: Uses Gemini cookies (`SECURE_1PSID`, `SECURE_1PSIDTS`) for Gemini API access and optional `API_KEY` for server authentication
- **API Endpoints**:
  - `GET /`: Health check endpoint
  - `GET /v1/models`: Lists available Gemini models in OpenAI format
  - `POST /v1/chat/completions`: Main chat completion endpoint (supports streaming)

### Key Features

- OpenAI-compatible chat completions API
- Streaming response support
- Image processing (base64 encoded images via temporary files)
- Markdown link correction for Google search results
- CORS enabled for web clients
- Docker containerization with uv package manager

## Coding Conventions

When making changes to this codebase, please adhere to the following principles:

- **Keep It Simple, Stupid (KISS):** Write code that is simple, straightforward, and easy to understand. Avoid introducing unnecessary complexity.
- **Don't Repeat Yourself (DRY):** Instead of duplicating code for similar functionalities, create generic, reusable functions. A good example is the `initProviderFilter` function in `scripts/settings.js`, which handles filtering logic for multiple providers in a unified way.
- **Centralize Configuration:** Group related configurations together to make the code easier to maintain and extend. For instance, the `filterConfigurations` array in `scripts/settings.js` centralizes all the settings for the provider-specific filters, making it easy to add new ones in the future.

## Development Commands

### Environment Setup
```bash
# Install dependencies with uv (recommended)
uv sync

# Or with pip
pip install fastapi uvicorn gemini-webapi

# Set up environment variables (copy from example)
cp .env.example .env
# Edit .env with your Gemini credentials
```

### Running the Server
```bash
# Development server with auto-reload
uvicorn main:app --reload --host 127.0.0.1 --port 8000

# Production server
uvicorn main:app --host 0.0.0.0 --port 8000

# Using uv
uv run uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

### Code Quality
```bash
# Lint and format with ruff
ruff check .
ruff format .
```

### Docker Commands
```bash
# Build and run with docker-compose
docker-compose up -d

# View logs
docker-compose logs

# Rebuild and restart
docker-compose up -d --build

# Stop services
docker-compose down
```

## Configuration

### Required Environment Variables
- `SECURE_1PSID`: Gemini cookie for authentication (obtained from browser dev tools)
- `SECURE_1PSIDTS`: Gemini cookie timestamp for authentication
- `API_KEY`: Optional server authentication key
- `ENABLE_THINKING`: Optional boolean to enable thinking content in responses (default: false)

### Code Style
- Uses ruff for linting and formatting
- Line length: 150 characters
- Tab-based indentation
- Double quotes for strings
- Ignores E501 (line length warnings due to custom 150 char limit)

## Model Mapping

The server maps OpenAI model names to Gemini models through `map_model_name()` function. It supports fuzzy matching and falls back to sensible defaults based on keywords (pro, flash, vision, etc.).

## Request Flow

1. Client sends OpenAI-compatible request to `/v1/chat/completions`
2. Server authenticates using optional API_KEY
3. Messages are converted from OpenAI format to conversation string
4. Images are decoded from base64 and saved to temporary files
5. Request is sent to Gemini via `gemini-webapi`
6. Response is processed, markdown corrected, and returned in OpenAI format
7. Temporary files are cleaned up

---
> Source: [zhiyu1998/Gemi2Api-Server](https://github.com/zhiyu1998/Gemi2Api-Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
