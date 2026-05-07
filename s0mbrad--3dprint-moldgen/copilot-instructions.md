## python-backend

> Python backend coding standards for moldgen/ files


# Python Backend Standards

- Python 3.11+, use `from __future__ import annotations` in all files
- Ruff: line-length=100, target=py311
- Type hints on all function signatures
- Use `StrEnum` for enums, `dataclass` for data containers, Pydantic `BaseModel` for API schemas
- Async preferred for API handlers and AI calls
- Singleton pattern via `__new__` for ToolRegistry, AIServiceManager, AgentMemoryManager
- Logging: `logger = logging.getLogger(__name__)` at module level
- Config: access via `moldgen.config.get_config()`, never hardcode values

---
> Source: [S0mbraD/3DPrint_MoldGen](https://github.com/S0mbraD/3DPrint_MoldGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
