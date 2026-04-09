## claudecode2openai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code CLI to OpenAI API conversion service** that provides OpenAI-compatible API endpoints while forwarding requests to Claude's Anthropic API. It's specifically designed to allow tools and applications expecting OpenAI API format to seamlessly work with Claude models.

## Architecture

### Core Components

- **FastAPI Application** (`main.py`): Main HTTP server with OpenAI-compatible endpoints
- **Configuration Management** (`src/config.py`): YAML-based configuration with model mapping support
- **Claude Manager** (`src/services/claude_manager.py`): Handles Claude API communication and format conversion
- **Supporting Services**:
  - `request_logger.py`: Request/response logging
  - `rate_limiter.py`: API rate limiting
  - `metrics.py`: Metrics collection and monitoring

### Request Flow

1. Client sends OpenAI-format request (e.g., `model: "gpt-4o"`) to `/v1/chat/completions`
2. Service maps OpenAI model name to corresponding Claude model (e.g., `gpt-4o` → `claude-3-5-sonnet-20241022`)
3. Request format is converted from OpenAI to Claude API format
4. Request is forwarded to Claude API
5. Claude response is converted back to OpenAI-compatible format
6. OpenAI-format response is returned to client

### Model Mapping System

The service uses a flexible model mapping system defined in `config.yaml`:
- OpenAI model names (e.g., `gpt-4o`, `gpt-4-turbo`) are mapped to Claude model IDs
- This allows seamless integration with existing OpenAI clients
- Model mappings can be customized in the configuration file

## Development Commands

### Environment Setup

```bash
# Using uv (recommended)
uv sync
uv run .

# Using pip
pip install -r requirements.txt
python main.py

# Using the startup script (automatic environment setup)
./start.sh           # Production mode
./start.sh --dev     # Development mode with hot reload
```

### Running the Service

```bash
# Direct execution
python main.py

# Module execution
python __main__.py

# Using uv
uv run .

# Development with hot reload
uvicorn main:app --reload
```

### Testing and Quality

```bash
# Run tests (if test files exist)
pytest

# Code formatting
black .

# Import sorting
isort .

# Type checking
mypy .

# Linting
flake8 .
```

## Configuration

### Required Environment Variables

Set this before running the service:

```bash
export CLAUDE_API_KEY="your-claude-api-key"

# Optional environment variables:
export PORT=8000          # Custom port (default: 8000)
export HOST=0.0.0.0       # Custom host (default: 0.0.0.0)  
export DEBUG=true         # Enable debug mode
```

### Configuration File

Edit `config.yaml` to customize:
- Server settings (host, port, CORS)  
- Claude API configuration (base_url, timeout, retry logic)
- Model mapping (OpenAI names → Claude model IDs)
- Monitoring and logging levels
- Rate limiting settings

Example model mapping:
```yaml
claude:
  models:
    - id: "claude-3-5-sonnet-20241022"
      name: "gpt-4o"  # Maps gpt-4o → claude-3-5-sonnet
      family: "claude-3.5"
```

### Port Configuration

Default port is 8000. Override with:
```bash
export PORT=9000
# or
PORT=9000 ./start.sh
```

## Model Management

### Adding New Model Mappings

1. Add model mapping to `config.yaml` under `claude.models`:
```yaml
- id: "claude-3-haiku-20240307"    # Claude model ID
  name: "gpt-3.5-turbo-instruct"   # OpenAI-compatible name
  family: "claude-3"                # Model family
```

2. The service will automatically:
   - Accept requests using the OpenAI name (`gpt-3.5-turbo-instruct`)
   - Map to the Claude model ID (`claude-3-haiku-20240307`) 
   - Convert request/response formats appropriately

### Model Routing Logic

- Exact model name matching (case-sensitive)
- Validates model exists in configuration before processing
- Returns clear error message for unsupported models
- All models route to Claude API (single backend)

## API Endpoints

### Core Endpoints
- `POST /v1/chat/completions` - Chat completions (OpenAI compatible, forwarded to Claude)
- `POST /v1/completions` - Text completions (converted to chat format)
- `GET /v1/models` - List available models (OpenAI format)

### Monitoring Endpoints
- `GET /health` - Health check with Claude API status
- `GET /models` - Model mapping details (custom endpoint)
- `GET /stats` - Usage statistics
- `GET /metrics` - Prometheus metrics

## Common Tasks

### Adding a New Model
1. Add model mapping to `claude.models` list in `config.yaml`
2. No code changes needed - mapping is automatic
3. Service restart not required (configuration is loaded dynamically)

### Debugging Request Issues
1. Enable request logging: set `monitoring.log_requests: true` in config
2. Check logs in `app.log`
3. Use `/health` endpoint to verify Claude API connectivity
4. Use `/models` endpoint to check model mappings
5. Test with simple cURL request to verify format conversion

### Performance Tuning
- Adjust Claude API `timeout` and `max_retries` in configuration
- Configure rate limiting in `rate_limit` section
- Monitor metrics via `/metrics` endpoint
- Claude API has its own rate limits - monitor response codes

## Code Style

The project uses:
- **Black** for code formatting (line length: 88)
- **isort** for import sorting
- **mypy** for type checking
- **flake8** for linting

Configuration is defined in `pyproject.toml`.

## Dependencies

Core dependencies:
- `fastapi` - Web framework
- `uvicorn` - ASGI server  
- `httpx` - HTTP client for Claude API communication
- `pydantic` - Data validation and configuration models
- `pyyaml` - Configuration file parsing

Development dependencies are defined in the `dev` dependency group.

## Key Implementation Details

### Format Conversion
- OpenAI → Claude: Separates system messages, adjusts parameter names
- Claude → OpenAI: Maps content arrays to simple text, converts usage tokens
- Stop reason mapping: `end_turn` → `stop`, `max_tokens` → `length`

### Error Handling
- Exponential backoff retry for server errors (5xx)
- Client errors (4xx) are not retried
- Health status tracking based on API response success
- Detailed error logging with request context

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kyne0116)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kyne0116)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
