## vllm-tuner

> This file contains guidelines for agentic coding assistants working on the vllm-tuner repository.

# AGENTS.md

This file contains guidelines for agentic coding assistants working on the vllm-tuner repository.

## Build, Lint, and Test Commands

### Installation
- **Install with dev dependencies**: `uv pip install -e ".[dev]"`
- **Install vLLM**: `uv pip install vllm` (required for integration tests)

### Code Quality
- **Format code**: `black src/ tests/`
- **Check linting**: `ruff check src/ tests/`
- **Auto-fix linting**: `ruff check src/ --fix`
- **Type checking**: `mypy src/`

### Testing
- **Run all tests**: `pytest`
- **Run specific test file**: `pytest tests/unit/test_config.py`
- **Run single test function**: `pytest tests/unit/test_config.py::test_gpu_config_default`
- **Run with coverage**: `pytest --cov=src tests/ --cov-report=html`
- **Run integration tests**: `pytest tests/integration/`
- **Run unit tests only**: `pytest tests/unit/`

## Code Style Guidelines

### Imports
Aim for clear, readable import statements. Guide: standard library first, then third-party, then local imports.

Consider blank lines between groups. Prefer relative imports within packages: `from ..config.models import TuningConfig` instead of absolute `from src.config.models import TuningConfig`.

```
# Group by: standard library, third-party, local
import asyncio
import logging
from pathlib import Path
from typing import Optional, Dict, Any

import httpx
import pynvml
from pydantic import BaseModel, Field

from ..config.models import TuningConfig
from .optimizer import VLLMOptimizer
```

### Formatting
- Line length: 100 characters
- Indentation: 4 spaces
- Docstrings: Triple quotes `"""` style, one-liner for simple functions
- String quotes: Single quotes unless string contains single quote

### Naming Conventions
- Classes: PascalCase (`GPUConfig`, `StudyManager`)
- Functions/variables: snake_case (`collect_all()`, `device_ids`)
- Constants: UPPER_SNAKE_CASE (`DEFAULT_RANGES`)
- Private members: Underscore prefix (`_prompts`, `_run_trial()`)
- Configuration classes: Suffix with `Config` (`WorkloadConfig`)
- Validation models: Suffix with `Model` (Pydantic handles this implicitly)

### Type Annotations
- All functions require type hints
- Prefer modern style `list[str]` over legacy `List[str]`
- Use `Optional[T]` for nullable return types
- Multiple return values: Use `tuple[type1, type2]`
- Dictionary with string keys: `dict[str, Any]` or `dict[str, specific_type]`
- Use `Any` sparingly; prefer specific types when possible

```
def collect_all(self) -> List[GPUStats]:
    """Collect metrics from all monitored GPUs."""

async def run_benchmark(
    self,
    prompts: List[str],
    metrics_tracker: VLLMMetricsTracker,
) -> Dict[str, Any]:
    """Run benchmark with provided prompts."""
```

### Error Handling
- Log errors with context before re-raising
- Use finally blocks for cleanup (especially async resources)
- Raise descriptive exceptions (`ValueError`, `RuntimeError`)
- Log the error traceback with `exc_info=True` for debugging

```
try:
    result = await some_async_operation()
except ConnectionError as e:
    logger.error(f"Connection failed: {e}", exc_info=True)
    raise
finally:
    await cleanup_resources()
```

### Async Patterns
- Use `async with` for context managers
- Create tasks with `asyncio.create_task()`, store in class attribute for cancellation
- Add type hint for async tasks: `Optional[asyncio.Task]`
- Handle `asyncio.CancelledError` in long-running loops

```
self.monitoring_task: Optional[asyncio.Task] = None
stop_event = asyncio.Event()

async def _monitor(self):
    while not stop_event.is_set():
        try:
            await collect()
        except asyncio.CancelledError:
            logger.info("Monitoring cancelled")
            raise

try:
    self.monitoring_task = asyncio.create_task(self._monitor())
finally:
    stop_event.set()
    if self.monitoring_task and not self.monitoring_task.done():
        self.monitoring_task.cancel()
```

### Safe Event Loop Handling
Check if event loop is running before using `asyncio.run()`:

```
try:
    loop = asyncio.get_running_loop()
except RuntimeError:
    loop = None

if loop and loop.is_running():
    result = await some_async_function()
else:
    result = asyncio.run(some_async_function())
```

### Testing Guidelines
- Descriptive test names: `test_function_name_scenario()`
- Use pytest fixtures for setup/teardown
- Unit tests in `tests/unit/`, integration in `tests/integration/`
- Mock external dependencies (vLLM server, GPU)
- Test both success and failure paths

```
def test_gpu_config_validation():
    """Test GPUConfig validates device_ids correctly."""
    config = GPUConfig(device_ids=[0, 1])
    assert config.count == 2

def test_weighted_objectives_invalid_sum():
    """Test weighted objectives must sum to 100."""
    with pytest.raises(ValidationError):
        WeightedObjectives(throughput=50, latency=50, memory=50)
```

### Configuration (Pydantic Models)
- Use Pydantic models with `Field()` for configuration
- Add `description` parameter for documentation
- Use `@field_validator` for multi-field validation
- Set `default_factory=list` for mutable defaults

```
class WorkloadConfig(BaseModel):
    max_tokens: int = Field(default=256, ge=1, description="Max output tokens per request")

@field_validator("throughput", "latency", "memory")
@classmethod
def validate_weights(cls, v: int) -> int:
    if v < 0 or v > 100:
        raise ValueError("Weights must be between 0 and 100")
    return v
```

### Logging
- Module-level logger: `logger = logging.getLogger(__name__)`
- Log levels: DEBUG (detailed flow), INFO (normal operation), WARNING (recoverable issues), ERROR (failures)
- Format: `%(asctime)s - %(name)s - %(levelname)s - %(message)s`
- Reduce noisy library logging: `logging.getLogger("httpx").setLevel(logging.WARNING)`

```
logger = logging.getLogger(__name__)

logger.info("Starting vLLM server...")
logger.warning("GPU {gpu_id} not available (max: {max_id})")
logger.error("Failed to initialize NVML: {e}", exc_info=True)
```

### File Structure
```
src/
├── __init__.py
├── cli/              # Typer CLI commands and entry point
├── config/           # Pydantic validation models (models.py, validation.py)
├── tuner/            # Optuna study manager and optimizer
├── vllm/             # vLLM launcher and telemetry parser
├── profiling/        # GPU collectors and vLLM metrics tracker
├── benchmarks/       # Workload loader and request generator
├── reporting/        # HTML reports and progress dashboard
└── optimization/     # Parameter search space definition

tests/
├── unit/             # Unit tests for individual modules
└── integration/      # Full workflow tests (requires vLLM server)
```

### Important Notes
- Line length of 100 is enforced by Black
- Type checking is strict: `disallow_untyped_defs = true` in mypy config
- Never commit secrets (API keys, passwords) to the repository
- vLLM server must be running for integration tests
- GPU access required for GPU profiling tests
- Study data is saved to `studies/<study_name>/` directories

### Performance Considerations
- Use async/await for I/O operations (HTTP, subprocess)
- Reuse GPU collectors across trials in study (initialize once, clear after each trial)
- Limit prompt dataset loading during development
- Use `pytest.mark.skip` for slow integration tests in CI

---
> Source: [jranaraki/vllm-tuner](https://github.com/jranaraki/vllm-tuner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
