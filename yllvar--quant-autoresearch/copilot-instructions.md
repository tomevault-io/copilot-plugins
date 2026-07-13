## quant-autoresearch

> This file provides guidelines for agents working in this repository.

# Agentic Coding Guidelines for Quant Autoresearch

This file provides guidelines for agents working in this repository.

## Project Overview

Quant Autoresearch is an autonomous quantitative strategy discovery framework using AI agents. It uses Python 3.12+, pytest for testing, and uv for package management.

---

## Build / Lint / Test Commands

### Running Tests

```bash
# Run all tests
pytest

# Run a single test file
pytest tests/unit/test_engine_opendev.py

# Run a single test function
pytest tests/unit/test_engine_opendev.py::test_engine_initialization

# Run tests with coverage
pytest --cov=src --cov-report=term-missing

# Run tests by marker
pytest -m "unit"
pytest -m "integration"

# Run tests in specific directory
pytest tests/unit/
pytest tests/integration/
```

### Package Management (uv)

```bash
# Install dependencies
uv sync

# Add a dependency
uv add <package>

# Add dev dependency
uv add --dev <package>

# Run Python scripts
uv run python cli.py run --iterations 10
```

### Type Checking

This project uses Python's typing system. Run type checking with:

```bash
# Install mypy first
uv add --dev mypy

# Run type checker
uv run mypy src/
```

---

## Code Style Guidelines

### General Principles

- Use Python 3.10+ compatible syntax (project uses 3.12.0)
- Keep lines under 120 characters
- Use 4 spaces for indentation (no tabs)
- Use meaningful variable and function names

### Imports

**Organize imports in this order (alphabetically within each group):**
1. Standard library (`os`, `sys`, `json`, `re`, `ast`, `subprocess`, `time`)
2. Third-party packages (`groq`, `dotenv`, `pytest`)
3. Local application imports (relative imports from `src/`)

```python
# Correct import order
import os
import json
import re
from typing import Dict, List, Optional, Tuple, Any

from groq import Groq
from dotenv import load_dotenv

from core.engine import QuantAutoresearchEngine
from safety.guard import SafetyLevel, ApprovalMode
from utils.logger import logger
```

### Type Annotations

- Use type hints for function arguments and return values
- Use `Dict`, `List`, `Optional`, `Tuple`, `Any` from `typing` module
- Prefer concrete types over `Any` when possible

```python
# Good
def process_data(data: Dict[str, int]) -> List[str]:
    ...

# Acceptable for complex types
async def run(max_iterations: int = 10) -> Optional[Dict[str, Any]]:
    ...
```

### Naming Conventions

- **Classes:** `PascalCase` (e.g., `QuantAutoresearchEngine`, `SafetyGuard`)
- **Functions/methods:** `snake_case` (e.g., `run_backtest`, `get_context_status`)
- **Constants:** `UPPER_SNAKE_CASE` (e.g., `MAX_CONTEXT_PERCENT`)
- **Private methods:** prefix with underscore (e.g., `_phase_thinking`)
- **Module names:** `snake_case` (e.g., `tool_registry.py`)

### Error Handling

- Use try/except blocks for operations that may fail
- Log errors with appropriate level (`logger.error`, `logger.warning`)
- Provide meaningful error messages
- Catch specific exceptions when possible

```python
# Good error handling
try:
    result = subprocess.run(["uv", "run", "python", script], 
                          capture_output=True, text=True)
except FileNotFoundError:
    logger.error("Python interpreter not found")
    return None
except Exception as e:
    logger.error(f"Execution failed: {e}")
    return None
```

### Docstrings

Use Google-style docstrings for classes and functions:

```python
def run_backtest_with_output() -> Tuple[float, float, int, float, Dict]:
    """Execute backtest and parse results.
    
    Returns:
        Tuple of (score, drawdown, trades, p_value, output_dict)
    """
    ...
```

### Class Structure

```python
class QuantAutoresearchEngine:
    """OPENDEV Enhanced ReAct Loop engine for Quant Autoresearch"""
    
    def __init__(self, safety_level: SafetyLevel = SafetyLevel.HIGH,
                 max_context_percent: int = 95):
        self.safety_level = safety_level
        self.max_context_percent = max_context_percent
        
        # Core components
        self.compactor = ContextCompactor(...)
    
    def run(self, max_iterations: int = 10):
        """Run the autonomous research loop."""
        ...
    
    def _phase_context_mgmt(self):
        """Phase 0: Adaptive Context Compaction."""
        ...
```

### Async/Await

- Use `async`/`await` for I/O-bound operations
- Use `asyncio.run()` to entry points
- Configure pytest with `asyncio_mode = auto` (already in pytest.ini)

```python
async def run(self, max_iterations: int = 10):
    for i in range(max_iterations):
        result = await self._some_async_call()
        ...

# Entry point
if __name__ == "__main__":
    import asyncio
    engine = QuantAutoresearchEngine()
    asyncio.run(engine.run())
```

### Testing Conventions

- Use pytest fixtures for setup/teardown
- Use `unittest.mock` for mocking external dependencies
- Name test files as `test_<module>.py`
- Name test functions as `test_<description>`

```python
@pytest.fixture
def engine(tmp_path):
    db_path = tmp_path / "test_playbook.db"
    with patch("core.engine.Groq"):
        engine = QuantAutoresearchEngine(db_path=str(db_path))
        return engine

def test_engine_initialization(engine):
    assert engine.safety_level == SafetyLevel.LOW
```

### File Organization

```
src/
├── core/           # Engine logic, backtester
├── safety/        # Guardrails, safety checks
├── models/         # LLM routing
├── memory/        # Pattern playbook
├── tools/         # Tool registry
├── prompts/       # System instructions
├── context/       # Context management
├── utils/         # Logger, telemetry, trackers
└── data/          # Data processing

tests/
├── unit/          # Unit tests
├── integration/   # Integration tests
├── regression/    # Regression tests
└── security/     # Security tests
```

### Configuration

- Environment variables go in `.env` (not committed)
- Template in `.env.example`
- Use `python-dotenv` for loading

---

## Key Architecture Patterns

### OPENDEV 6-Phase Cycle

1. **Phase 0 - Context Management:** Adaptive Context Compaction (ACC)
2. **Phase 1 - Thinking:** Use separate model for reasoning
3. **Phase 2 - Action Selection:** Primary model selects tools
4. **Phase 3 - Doom-Loop Detection:** Fingerprint checking
5. **Phase 4 - Execution:** 5-layer defense-in-depth
6. **Phase 5 - Observation:** Output optimization and playbook storage

### Safety Layers

1. Prompt Guardrails
2. Schema Gating
3. Runtime Approval
4. Tool Validation
5. Look-Ahead Scanner (AST-based)

---

## Notes

- All tests must pass before merging (run `pytest` to verify)
- Use type hints where practical
- Keep functions focused and small
- Document complex logic with comments
- Always log important operations

---
> Source: [yllvar/Quant-Autoresearch](https://github.com/yllvar/Quant-Autoresearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
