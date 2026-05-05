## claude-router

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Claude Router proxy server that connects local Ollama servers with GPT-OSS models to provide Claude Code functionality. The Python FastAPI application acts as a proxy between Claude Code and local Ollama instances.

## Architecture

### Core Components

- **main.py**: FastAPI proxy server that handles Anthropic Messages API compatibility
  - Converts Claude message format to Ollama prompts 
  - Streams responses from Ollama back to Claude Code
  - Handles `/v1/messages`, `/health`, and root endpoints

- **Docker Setup**: Containerized deployment using Python 3.11-slim base
  - Copies `.claude/settings.local.json` configuration into container
  - Exposes port 4000 for proxy communication

- **Claude Code Integration**: Uses `.claude/settings.local.json` to:
  - Set `ANTHROPIC_BASE_URL` to `http://localhost:4000`
  - Enable all tools with `allowedTools: ["*"]`
  - Configure trust and onboarding flags

### Message Flow

1. Claude Code sends requests to localhost:4000 (proxy)
2. Proxy converts Anthropic message format to Ollama prompt format
3. Proxy forwards to local Ollama server at localhost:11434
4. Ollama responses are streamed back through proxy to Claude Code

## Development Commands

### Running the Proxy Server

```bash
# Set permissions and run setup script
chmod +x run.sh
./run.sh
```

This will:
1. Generate `.claude/settings.local.json` configuration
2. Build Docker image as `claude-router`
3. Run container with port mapping 4000:4000

### Direct Python Execution

```bash
# Install dependencies
pip install -r requirements.txt

# Run directly (alternative to Docker)
python main.py
```

### Docker Commands

```bash
# Build image
docker build -t claude-router .

# Run container
docker run -it --rm -p 4000:4000 claude-router
```

## Prerequisites

- Ollama installed and running locally
- GPT-OSS model pulled: `ollama pull gpt-oss:20b`
- Docker installed for containerized deployment

## Configuration

### Environment Variables

- `MODEL_NAME`: Ollama model to use (default: "gpt-oss:20b")
- `OLLAMA_URL`: Ollama API endpoint (default: "http://localhost:11434/api/generate")

### Claude Code Settings

The proxy automatically generates `.claude/settings.local.json` with:
- Proxy server URL configuration
- All tools enabled
- Trust and onboarding settings pre-configured

### Test

Every test must be success.
- The test folder is `./test`.

---
> Source: [say828/claude-router](https://github.com/say828/claude-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
