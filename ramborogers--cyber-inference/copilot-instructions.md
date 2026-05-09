## cyber-inference

> This document provides essential information for agents working on the Cyber-Inference codebase. It covers commands, patterns, conventions, and gotchas observed in the project.

# AGENTS.md - Cyber-Inference Development Guide

This document provides essential information for agents working on the Cyber-Inference codebase. It covers commands, patterns, conventions, and gotchas observed in the project.

## Essential Commands

### Development Setup
```bash
# Install dependencies (uses uv package manager)
uv sync

# Initialize directories and database
uv run cyber-inference init

# Start development server
uv run cyber-inference serve --reload

# Install/update llama.cpp server binary
uv run cyber-inference install-llama
```

### Model Management
```bash
# Download a model from HuggingFace
uv run cyber-inference download-model <hf-repo-id> [--filename <specific-file>]

# List downloaded models
uv run cyber-inference list-models
```

### Testing & Code Quality
```bash
# Run tests (pytest with asyncio support)
uv run pytest

# Run linting (ruff)
uv run ruff check .

# Run type checking (mypy)
uv run mypy src/
```

### Production Commands
```bash
# Start production server
uv run cyber-inference serve --host 0.0.0.0 --port 8337

# Docker deployment
docker-compose up -d                    # CPU
docker-compose -f docker-compose.nvidia.yml up -d  # NVIDIA GPU
docker-compose -f docker-compose.jetson.yml up -d  # NVIDIA Jetson
```

## Code Organization and Structure

### Directory Layout
```
src/cyber_inference/
├── main.py              # FastAPI app initialization, lifespan management
├── cli.py               # Typer CLI commands
├── core/                # Core functionality
│   ├── config.py        # Pydantic settings with env var support
│   ├── database.py      # SQLAlchemy async setup
│   ├── logging.py       # Rich logging configuration
│   └── auth.py          # JWT authentication
├── services/            # Business logic services
│   ├── process_manager.py    # Manages llama.cpp server processes
│   ├── model_manager.py      # Model download/storage management
│   ├── resource_monitor.py   # System resource monitoring
│   ├── auto_loader.py         # Automatic model loading/unloading
│   └── llama_installer.py     # llama.cpp binary installation
├── models/              # Data models and schemas
│   ├── db_models.py     # SQLAlchemy ORM models
│   └── schemas.py       # Pydantic API schemas (OpenAI-compatible)
├── api/                 # API route handlers
│   ├── v1.py           # OpenAI-compatible endpoints
│   ├── admin.py        # Admin/management endpoints
│   ├── web.py          # Web GUI routes
│   └── websocket.py    # Real-time updates (logs, status)
└── web/                 # Web interface
    ├── templates/      # Jinja2 HTML templates
    └── static/         # CSS/JS assets (currently empty)

tests/                   # pytest test suite
scripts/                # Utility scripts
├── test_embeddings.sh   # cURL test for embeddings endpoint
└── test_model_inference.py  # Python integration test suite
```

### Key Architecture Patterns

- **Async-first**: All database operations, HTTP calls, and long-running tasks use async/await
- **Service layer**: Business logic separated into service classes (ProcessManager, ModelManager, etc.)
- **Lifespan management**: FastAPI lifespan events handle startup/shutdown of global services
- **Global instances**: Services like ProcessManager are global singletons accessed via getter functions
- **SQLAlchemy async**: Database operations use AsyncSession throughout
- **Pydantic settings**: Configuration uses Pydantic v2 with environment variable support

## Naming Conventions and Style Patterns

### Code Style
- **Python version**: 3.12+
- **Line length**: 100 characters (ruff config)
- **Linting**: ruff with E, F, I, N, W, UP rules (ignores E501 line length)
- **Type hints**: Extensive use of type hints, mypy strict mode enabled
- **Imports**: Absolute imports within the package, standard library first, then third-party

### Naming Patterns
- **Functions/variables**: snake_case (`get_settings()`, `process_manager`)
- **Classes**: PascalCase (`ProcessManager`, `ModelSession`)
- **Constants**: UPPER_CASE (minimal usage)
- **Files**: snake_case (`model_manager.py`, `db_models.py`)
- **Database tables**: snake_case (`model_sessions`, `configurations`)
- **API endpoints**: RESTful with kebab-case in URLs but snake_case in code

### SQLAlchemy Patterns
- **Table names**: snake_case (`models`, `model_sessions`)
- **Column names**: snake_case (`model_id`, `is_downloaded`)
- **Relationships**: Explicit foreign keys and back_populates
- **Timestamps**: `created_at`, `updated_at`, `last_used_at` with timezone-aware datetime

### FastAPI Patterns
- **Router organization**: Separate routers for different API versions/features
- **Dependency injection**: Global service getters rather than Depends injection
- **Error handling**: HTTPException with status codes
- **Response models**: Pydantic models for all API responses
- **Middleware**: CORS enabled, custom logging

## Testing Approach and Patterns

### Test Framework
- **pytest** with **pytest-asyncio** for async test support
- **Test discovery**: `pythonpath = ["src"]` in pytest.ini
- **Async mode**: `asyncio_mode = "auto"` (auto-detect async tests)

### Test Structure
```python
@pytest.mark.asyncio
async def test_endpoint():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.get("/endpoint")
        assert response.status_code == 200
```

### Test Categories
- **Integration tests**: API endpoint testing with httpx AsyncClient
- **Unit tests**: Individual service/component testing
- **Error handling**: Tests for various failure scenarios
- **Async operations**: Tests for database operations, model loading, etc.

## Important Gotchas and Non-Obvious Patterns

### Global Service Instances
Services are initialized globally in `main.py` lifespan and accessed via getters:
```python
def get_process_manager() -> ProcessManager:
    if process_manager is None:
        raise RuntimeError("Process manager not initialized")
    return process_manager
```
This pattern ensures services are available during request handling but prevents circular imports.

### Database Configuration Override
Settings can be overridden at runtime and stored in database via `/admin/config` endpoints. The `apply_db_config_overrides()` function merges database values with environment settings on startup.

### Model Loading/Unloading
- Models load automatically on first API request
- Auto-loader service unloads idle models after `model_idle_timeout` seconds
- Each model runs in separate llama.cpp server process on unique port
- GPU layers configured via `llama_gpu_layers` (-1 = auto-detect)

### Environment Variables
All settings use `CYBER_INFERENCE_` prefix. Key variables:
- `CYBER_INFERENCE_DATA_DIR`: Data/logs/config storage
- `CYBER_INFERENCE_MODELS_DIR`: Model file storage
- `CYBER_INFERENCE_ADMIN_PASSWORD`: Optional admin authentication
- `CYBER_INFERENCE_LLAMA_GPU_LAYERS`: GPU acceleration config

### Docker Considerations
- Multi-stage Dockerfile with uv for fast Python package management
- Separate compose files for CPU, NVIDIA GPU, and Jetson platforms
- Volume mounts required for data persistence (`/app/data`, `/app/models`)
- Health checks and proper user permissions (non-root)

### WebSocket Logging
Real-time log streaming uses WebSocket connections. Logger setup includes WebSocket handler for live updates in the web UI.

### Security Model
- Admin endpoints require JWT tokens when `admin_password` is set
- Bearer token extraction from `Authorization` header or cookies
- Optional authentication - disabled by default for ease of use

### Resource Management
- Resource monitor tracks CPU, memory, GPU usage
- Memory limits enforced via `max_memory_percent`
- Automatic model unloading prevents resource exhaustion

### Build System
- Uses `hatchling` build backend
- `src/` layout (not flat) specified in `pyproject.toml`
- CLI entry point: `cyber-inference = "cyber_inference.cli:app"`

## Development Workflow

1. **Setup**: `uv sync` to install dependencies
2. **Initialize**: `uv run cyber-inference init` for directories and database
3. **Develop**: `uv run cyber-inference serve --reload` for development
4. **Test**: `uv run pytest` and `uv run ruff check .` before commits
5. **Deploy**: Use appropriate docker-compose file for target platform

## Common Issues

- **Port conflicts**: llama.cpp servers start on `llama_server_base_port` (8338) and increment
- **Model loading failures**: Check logs for llama.cpp errors, ensure sufficient RAM/GPU memory
- **Database issues**: Run `init` command to create tables if missing
- **Import errors**: Always use `uv run` to ensure proper Python path and dependencies</content>
<parameter name="file_path">AGENTS.md

---
> Source: [RamboRogers/cyber-inference](https://github.com/RamboRogers/cyber-inference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
