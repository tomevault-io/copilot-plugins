## cloud-vm-mcp-py

> MCP server for managing VMs across cloud providers (AWS, Azure).

# AGENTS.md

MCP server for managing VMs across cloud providers (AWS, Azure).
Python 3.10+, uv package manager, FastMCP framework, Pydantic models.

## Build & Run

```bash
uv sync                          # Install all dependencies
uv run vm-mcp                    # Start the MCP server
uv run python cli.py list        # CLI: list VMs
```

## Lint, Format, Type Check

```bash
uv run ruff check .              # Lint (auto-fixes with --fix by default, fix=true in config)
uv run ruff format .             # Format
uv run ty check .                # Type check (Astral's ty, scoped to vm_mcp/)
```

Pre-commit hooks run `uv-lock` sync and `ty check` automatically.

## Tests

```bash
uv run pytest tests/ -v                                          # All tests
uv run pytest tests/test_config.py -v                            # Single file
uv run pytest tests/test_config.py::TestConfigLoader -v          # Single class
uv run pytest tests/test_config.py::TestConfigLoader::test_load_valid_config -v  # Single test
uv run pytest -k "test_load_valid" -v                            # By name pattern
uv run pytest tests/ --cov=vm_mcp --cov-report=html              # With coverage
```

Test stack: pytest + pytest-asyncio + pytest-mock. Async tests use `@pytest.mark.asyncio`.
Tests are class-based (e.g., `class TestConfigLoader:`). Fixtures in `tests/conftest.py`.

## Project Structure

```
vm_mcp/
├── __init__.py          # Exports main() and mcp
├── __main__.py          # python -m vm_mcp entry point
├── config.py            # ConfigLoader with file watching (hot-reload, watchdog)
├── mcp.py               # FastMCP server, all MCP tool definitions
├── model/
│   ├── credentials.py   # AWS/Azure credential Pydantic models
│   └── vm.py            # VMState, VMInfo, VMDetails, VMError models
└── providers/
    ├── base.py           # Abstract BaseProvider (ABC)
    ├── aws.py            # AWS EC2 provider
    ├── azure.py          # Azure Compute provider
    └── registry.py       # Provider factory/lookup
```

## Architecture

- **Provider pattern:** `BaseProvider` (ABC) → `AWSProvider`, `AzureProvider`. Add new providers by subclassing `BaseProvider`.
- **Composite VM ID:** `{provider}:{tenant_alias}:{region}:{instance_id}` — universal identifier across providers.
- **Config hot-reload:** `ConfigLoader` uses watchdog to watch `providers.yaml`. Disable with `enable_watch=False` for testing.
- **Sync-to-async bridge:** Cloud SDK calls (boto3, Azure) are synchronous; wrap with `asyncio.to_thread()`.
- **MCP tool output:** All tools return YAML strings via `yaml.dump()`. Responses use `{"status": "success"|"error"|"partial", ...}`.

## Code Style

### Imports

Ordered by: stdlib → third-party → local. Enforced by Ruff `extend-select = ["I"]`.
Use parenthesized multi-line imports with trailing commas:

```python
from vm_mcp.model.credentials import (
    AWSAccount,
    AWSConfig,
    ProvidersConfig,
)
```

### Naming

- `snake_case`: functions, methods, variables, parameters, modules
- `PascalCase`: classes (`ConfigLoader`, `AWSProvider`, `VMInfo`)
- `UPPER_SNAKE_CASE`: module-level constants (`PER_REQUEST_TIMEOUT`)
- `_single_underscore`: private/protected members (`_config`, `_get_client`)

### Types

- Use Python 3.10+ syntax: `str | None`, `list[str]`, `tuple[bool, str]`, `dict[str, Any]`
- Do NOT use `Optional[str]`, `List[str]`, `Tuple[...]`, `Dict[...]`
- Use Pydantic `BaseModel` for all data models (not dataclasses or TypedDict)
- Avoid `Any` — use only when SDK types are genuinely opaque (e.g., boto3 clients)
- Add `# type: ignore` only when ty cannot infer the type, not to silence real errors
- Abstract methods use `...` (Ellipsis) as body, not `pass` or `raise NotImplementedError`

### Error Handling

- Always use `logging.exception()` to log exceptions (captures full traceback automatically)
- Do NOT use `print()` or custom log messages for errors — use structured logging
- MCP tools: wrap in try/except, return YAML error responses, never raise to the framework
- Catch specific exceptions when possible (e.g., `ClientError` for AWS), fall back to `Exception`

```python
try:
    result = await asyncio.to_thread(sync_call)
except ClientError as e:
    if e.response["Error"]["Code"] != "InvalidInstanceID.NotFound":
        raise
except Exception:
    logger.exception("Failed to list VMs from provider %s", provider.provider_name)
```

### Docstrings

Google style. Module and class docstrings are single-line. Function docstrings use `Args:` and `Returns:`:

```python
def stop_vm(self, vm_id: str, force: bool = False) -> tuple[bool, str]:
    """Stop a VM.

    Args:
        vm_id: The instance ID (not composite ID)
        force: Force stop without graceful shutdown

    Returns:
        Tuple of (success, message)
    """
```

### Async Patterns

- Wrap synchronous SDK calls with `asyncio.to_thread()`
- Use `asyncio.wait_for()` for timeout enforcement
- Use `asyncio.gather()` for parallel provider queries
- Define inner sync functions inside async methods for the thread bridge:

```python
async def _list_vms_in_region(self, region: str) -> list[VMInfo]:
    def _describe():
        # synchronous boto3 code here
        ...
    instances = await asyncio.to_thread(_describe)
```

### Module Conventions

- Define `__all__` in all `__init__.py` files
- Use global singleton pattern with getter/reset functions (see `config.py`)
- Provider registry pattern for provider lookup by name

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `PROVIDERS_CONFIG_PATH` | Yes | Path to `providers.yaml` credentials file |
| `MCP_TRANSPORT` | No | Transport: `stdio` (default), `sse`, `streamable-http` |

## Copilot Instructions (from .github/copilot-instructions.md)

- Code should be simple, clear, and type-safe
- Use `uv run ty check` and `uv run ruff check` to verify
- Always use `logging.exception()` for error logging with full traceback
- MCP reference docs: https://modelcontextprotocol.io/quickstart/server

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sondt2709) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
