## mini-cc

> Guidance for agentic coding agents working in this repository.

# AGENTS.md

Guidance for agentic coding agents working in this repository.

## Project Overview

Mini Claude Code (`mini_cc`) is a lightweight multi-agent collaborative coding assistant CLI built in pure Python. It uses an async agent loop to stream LLM responses, execute tool calls (file read/write/edit, bash, glob, grep), and render results via a Textual TUI or prompt_toolkit REPL.

**Target platform:** Linux/WSL only.

## Repository Structure

```
src/mini_cc/
├── __init__.py              # Package version
├── __main__.py              # Entry point
├── cli.py                   # Typer CLI (tui / chat commands)
├── repl.py                  # REPL loop, engine factory, event rendering
├── agent/                   # Sub-agent orchestration, worktree management
├── context/                 # System prompt assembly + tool-use context
├── providers/               # LLM provider Protocol + OpenAI implementation
├── query_engine/            # Agent loop engine + state/event data models
├── task/                    # Task queue, dependency tracking, completion events
├── tool_executor/           # Concurrent/sequential tool execution
├── tools/                   # Individual tool implementations
└── tui/                     # Textual TUI (app, screens, widgets)

tests/                       # Mirrors src/mini_cc/ structure
├── test_repl.py
├── agent/
├── context/
├── query_engine/
├── task/
├── tool_executor/
├── tools/
└── tui/

docs/                        # Design documents (in Chinese)
```

## Build, Lint, and Test Commands

All commands use `uv run` to execute within the managed virtual environment.

```bash
uv sync                                    # Install all dependencies

# Lint & Format
uv run ruff check .                        # Lint (excludes tests/)
uv run ruff check . --fix                  # Lint with auto-fix
uv run ruff format --check .               # Format check only
uv run ruff format .                       # Format (write changes)

# Type check (mypy strict; tests are excluded via pyproject.toml override)
uv run mypy .

# Tests
uv run pytest                              # Run all tests
uv run pytest tests/test_repl.py           # Run a single test file
uv run pytest tests/test_repl.py::test_bar # Run a single test function
uv run pytest tests/test_repl.py::test_bar -v  # Single test, verbose
uv run pytest -k "pattern"                 # Run tests matching keyword
uv run pytest tests/tools/                 # Run all tests in a directory

# Git hooks (run once after clone)
uv run pre-commit install
uv run pre-commit install --hook-type commit-msg
```

## Code Style Guidelines

### Python Version & Formatting

- **Target Python version:** 3.11+
- **Line length:** 120 characters
- **Formatter:** `ruff format` (Black-compatible)
- **Every source file** starts with `from __future__ import annotations`

### Lint Rules (Ruff)

Enabled: `E`, `F`, `I`, `W`, `N`, `UP`. Note: ruff lint excludes `tests/**`.

### Imports

- Follow isort conventions (Ruff rule `I`).
- Three groups separated by blank lines: stdlib → third-party → local.
- Use absolute imports from the package root: `from mini_cc.query_engine.state import Message`.
- No relative imports anywhere in the codebase.
- `collections.abc` for `AsyncGenerator`, `Callable` (not `typing`).

### Type Annotations (mypy strict mode)

- All functions must have complete type annotations for parameters and return types.
- Use `str | None` — never `Optional[str]` (UP rule enforces modern union syntax).
- Use lowercase generics: `list[str]`, `dict[str, Any]` — never `List`, `Dict`.
- `Any` is acceptable only in `**kwargs: Any` and API payload dicts.
- Type aliases at module level in PascalCase: `Event = TextDelta | ToolCallStart | ...`
- `@runtime_checkable` Protocol classes for interfaces.

### Data Models

- **Pydantic `BaseModel`** for API-serializable data with validation (`ToolCall`, `Message`, `QueryState`, `Task`, tool input schemas).
- **`@dataclass`** for internal/event types and tracking structs. Use `frozen=True` for immutable config.
- **`StrEnum`** for enumeration types (e.g., `Role`, `AgentStatus`, `TaskStatus`).
- **Plain classes** for stateful services (`QueryEngine`, `StreamingToolExecutor`, `TaskService`).

### Naming Conventions

| Category | Convention | Example |
|----------|-----------|---------|
| Modules/packages | `snake_case` | `query_engine`, `tool_executor` |
| Classes | `PascalCase` | `QueryEngine`, `FileReadInput` |
| Functions/methods | `snake_case` | `submit_message`, `collect_tool_calls` |
| Constants | `UPPER_SNAKE_CASE` | `PLAN_MODE`, `BUILD_MODE` |
| Private members | `_` prefix | `_query_loop`, `_SAFE_TOOL_NAMES` |
| Type aliases | `PascalCase` | `Event`, `StreamFn` |

### Error Handling

- Tool errors are returned via `ToolResult(error="...", success=False)` — not raised exceptions.
- Catch specific exception types: `UnicodeDecodeError`, `OSError`, `json.JSONDecodeError`, `FileNotFoundError`, `subprocess.TimeoutExpired`.
- Never use bare `except:`.
- Use `raise ... from err` to preserve exception chains when re-raising.
- CLI exits with `sys.exit(1)` and a descriptive error message for fatal config errors.

### General Python Style

- Use f-strings for string formatting (enforced by UP rule).
- Use `pathlib.Path` exclusively — never `os.path`.
- UI-facing strings are in Chinese (tool descriptions, error messages, status bar).
- Define `__all__` in every `__init__.py` for public modules.
- No trailing whitespace; files end with a newline (pre-commit enforced).
- Do not add comments unless explicitly requested.

## Testing Conventions

- **Framework:** pytest with `pytest-asyncio` (async mode: `auto` in `pyproject.toml`).
- **File naming:** `test_<module>.py` mirroring source structure.
- **No `@pytest.mark.asyncio` decorator** needed — auto mode handles it.
- **Test classes** group related scenarios: `TestBash`, `TestBaseTool`, `TestQueryEngineTextOnly`.
- **Assertions:** plain `assert` statements only — no `unittest` assert helpers.
- **Fixtures:** use pytest `tmp_path` for filesystem tests. No `conftest.py` — helpers are local to each file with `_` prefix (`_DummyTool`, `_make_ctx`).
- **Async test pattern:** `[e async for e in stream]` list comprehension.

## Commit Convention

Follows [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `ci`

Use `cz commit` for interactive generation. Non-compliant messages are rejected by the commit-msg hook.

## CI Pipeline

GitHub Actions runs on every push/PR to `main` with Python 3.11 and 3.12:

1. `uv sync` — install dependencies
2. `uv run ruff check .` — lint
3. `uv run mypy .` — type check
4. `uv run pytest` — run tests

**All checks must pass before merging.**

## Key Design Decisions

- **Agent Loop:** Async streaming with event state machine (`TextDelta`, `ToolCallStart/Delta/End`, `ToolResultEvent`). See `docs/agent-loop/`.
- **Tool execution:** Safe tools (`file_read`, `glob`, `grep`) run concurrently; unsafe tools run sequentially. See `docs/tools/`.
- **Context:** System prompt assembled from static markdown + dynamic env info + optional AGENTS.md. See `docs/context/`.
- **Sub-agents:** `AgentManager` orchestrates `SubAgent` instances with git worktrees for isolation. See `src/mini_cc/agent/`.
- **Task system:** `TaskService` manages task queues with dependency tracking and completion events. See `src/mini_cc/task/`.

---
> Source: [liver0377/mini_cc](https://github.com/liver0377/mini_cc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
