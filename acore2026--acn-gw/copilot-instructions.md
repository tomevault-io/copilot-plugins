## acn-gw

> > This file provides guidelines for AI coding agents working on the Agent GW codebase.

# AGENTS.md - Coding Guidelines for Agent GW

> This file provides guidelines for AI coding agents working on the Agent GW codebase.

## Project Overview

Agent GW is a Python backend application with three functional entities:
- **ARF** (Agent Repository Function): HTTP REST API on port 9001 (FastAPI)
- **ACF** (Agent Communication Function): WebSocket server on port 9002
- **MOQT Relay**: MOQT protocol server on port 9003 (QUIC transport)

## Build/Lint/Test Commands

### Setup
```bash
# Initial setup (creates venv + installs deps)
./setup.sh

# Or manually:
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Running Tests
```bash
# Run all tests
python run_tests.py
pytest

# Run specific test categories
python run_tests.py unit
python run_tests.py integration

# Run specific test modules
python run_tests.py models      # tests/test_models.py
python run_tests.py arf         # tests/test_arf_api.py
python run_tests.py acf         # tests/test_acf_server.py
python run_tests.py moqt        # tests/test_moqt_relay.py

# Run single test file
pytest tests/test_models.py -v

# Run single test
pytest tests/test_models.py::TestAgentModel::test_create_agent -v

# Run with coverage
pytest --cov=. --cov-report=html --cov-report=term
```

### Running the Application
```bash
# Start all services
python3 agent_gw.py

# Start individual services
python -m agent_gw.arf_server      # ARF only (port 9001)
python -m agent_gw.acf_server      # ACF only (port 9002)
python3 agent_gw.py                # Full stack including MOQT Relay
```

### Linting
```bash
flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
```

### Maintenance
```bash
rm -rf venv .pytest_cache .test_integration_cache agent_gw/.relay_cache
find . -name '__pycache__' -type d -prune -exec rm -rf {} +
```

## Code Style Guidelines

### Python Version
- Python 3.8+ required
- Use type hints where appropriate (from typing import ...)

### Imports
```python
# 1. Standard library imports first
import asyncio
import json
from datetime import datetime
from typing import Dict, List, Optional

# 2. Third-party imports second
import websockets
from fastapi import FastAPI, HTTPException
from sqlalchemy import create_engine, Column

# 3. Local imports last
from models import Agent, Task
from logger_config import arf_logger
```

### Naming Conventions
- **Variables/Functions**: snake_case (`agent_id`, `handle_message()`)
- **Classes**: PascalCase (`Agent`, `ACFServer`, `ClientSession`)
- **Constants**: UPPER_CASE (`IDM_URL`, `DEFAULT_PORT`)
- **Private members**: _leading_underscore (`_internal_method()`)
- **Database models**: Singular nouns (`Agent`, `Task`, `Track`)

### File Structure
```python
#!/usr/bin/env python3
"""
Module docstring describing purpose.
Brief description of what this module does.
"""

# Imports grouped as described above

# Module-level constants
IDM_URL = "http://10.0.18.210:9020/idm/v1/vc-verifications"

# Classes and functions
class MyClass:
    """Class docstring."""
    pass

# Main entry point (if applicable)
if __name__ == '__main__':
    pass
```

### Type Hints
- Use type hints for function signatures
- Use `Optional[Type]` for nullable values
- Use `List[Type]`, `Dict[KeyType, ValueType]` for collections
- Use dataclasses with `@dataclass` decorator for data structures

### Error Handling
```python
# Always use try/except with specific exceptions
try:
    result = await process_data()
except json.JSONDecodeError:
    logger.info("Invalid JSON received")
except websockets.ConnectionClosed:
    logger.info("Connection closed")
except Exception as e:
    logger.info(f"Unexpected error: {e}")
    raise HTTPException(status_code=500, detail=str(e))
```

### Logging
- Use module-specific loggers from `logger_config.py`
- Available loggers: `arf_logger`, `acf_logger`, `moqt_logger`, `main_logger`
- Use `.info()` for normal operations, not `.error()` unless critical
- Log format: `logger.info(f"Descriptive message: {variable}")`

### Database Operations
```python
# Always use context manager or explicit close
db = get_db()
try:
    agent = db.query(Agent).filter(Agent.agent_id == agent_id).first()
    db.commit()
finally:
    db.close()
```

### Testing Guidelines
- Test files: `tests/test_<module>.py`
- Test classes: `Test<Feature>` (e.g., `TestAgentModel`)
- Test methods: `test_<description>` (e.g., `test_create_agent`)
- Use fixtures in `conftest.py` for shared setup
- Use `pytest` markers and parametrization where appropriate
- Mock external dependencies (HTTP calls, WebSockets)

### Code Formatting
- Indentation: 4 spaces (no tabs)
- Line length: Keep under 100 characters where possible
- Blank lines: 2 between top-level functions/classes, 1 within classes
- String quotes: Use single quotes for strings, double for docstrings
- Trailing commas in multi-line collections

### WebSocket Message Format
```python
{
    "type": "MESSAGE_TYPE",
    "timestamp": datetime.utcnow().isoformat() + 'Z',
    "payload": {
        "src_agent_id": "...",
        "dst_agent_id": "...",
        # ...
    }
}
```

### FastAPI Endpoints
```python
@app.post("/path/to/endpoint")
async def endpoint_name(request: Request):
    """Brief description of what this endpoint does."""
    try:
        data = await request.json()
        # Process...
        return JSONResponse(status_code=200, content={"status": "OK"})
    except Exception as e:
        logger.info(f"Error: {e}")
        raise HTTPException(status_code=500, detail=str(e))
```

## Project Structure
```
acn_gw/
├── moqt/                   # MOQT protocol implementation
│   ├── encoding/          # VarInt, KV pairs, Location
│   ├── messages/          # Control and data messages
│   └── transport/         # QUIC transport layer
├── tests/                 # Test suite
│   ├── conftest.py        # Shared fixtures
│   └── test_*.py          # Test files
├── models.py              # SQLAlchemy models (Agent, Task, Track)
├── arf_server.py          # ARF HTTP server
├── acf_server.py          # ACF WebSocket server
├── agent_gw.py           # Repository entry point (runs all services)
├── logger_config.py       # Logging configuration
├── run_tests.py           # Test runner script
├── requirements.txt       # Dependencies
└── setup.sh               # Environment setup
```

## Dependencies
Key packages used:
- `fastapi`, `uvicorn` - HTTP server
- `websockets` - WebSocket support
- `sqlalchemy`, `aiosqlite` - Database ORM
- `pydantic` - Data validation
- `aioquic` - QUIC transport for MOQT
- `pytest`, `pytest-asyncio`, `pytest-cov` - Testing
- `httpx` - HTTP client for testing

## Important Notes
- Database: SQLite (`agent_gw.db`), auto-created on startup
- Virtual environment required: `venv/` directory
- Logs: Written to `logs/` directory
- Ports: 9001 (ARF), 9002 (ACF), 9003 (MOQT)
- IDM integration: ARF forwards VC verification to external IDM service

---
> Source: [acore2026/acn_gw](https://github.com/acore2026/acn_gw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
