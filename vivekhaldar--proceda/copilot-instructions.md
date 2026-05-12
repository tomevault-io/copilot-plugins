## proceda

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Proceda?

Proceda is a terminal-first Python SDK for turning Standard Operating Procedures (SOPs) into runnable agents with built-in human oversight. Users write a `SKILL.md` file (markdown with YAML frontmatter and `### Step N:` headings), attach tools via MCP servers, and run it from the terminal.

## Build & Development Commands

```bash
uv sync --extra dev          # Install dependencies
make install-hooks           # Install pre-commit hooks (ruff format, ruff check, pytest, ty)

make test                    # Run pytest (uv run pytest -q)
make lint                    # Run ruff linter
make format                  # Run ruff formatter
make typecheck               # Run ty type checker (uvx ty check src/)
make check                   # Run all of the above

uv run pytest tests/test_skills/test_parser.py -q          # Run a single test file
uv run pytest tests/test_skills/test_parser.py::test_name  # Run a single test
uv run pytest -k "keyword" -q                              # Run tests matching keyword
```

Pre-commit hooks run ruff format, ruff check (with --fix --exit-non-zero-on-fix), pytest, and ty on every commit. Tests are fast (~1s).

## Architecture

The runtime follows this flow: **CLI/TUI/SDK -> Agent -> Runtime -> Executor -> LLM + MCP Tools**, with structured `RunEvent`s emitted at every step.

### Core concepts (four public types)

- **`Skill`** (`skill.py`) — Parsed `SKILL.md`: metadata, steps with markers (`[APPROVAL REQUIRED]`, `[PRE-APPROVAL REQUIRED]`, `[OPTIONAL]`), raw content.
- **`Agent`** (`agent.py`) — High-level API. `Agent.from_path()` loads a skill; `.run()` executes synchronously; `.run_stream()` yields events.
- **`RunSession`** (`session.py`) — Mutable state for one execution: status, current step, messages, pending approvals/clarifications, tool results.
- **`RunEvent`** (`events.py`) — Structured event record. Every runtime transition emits one. Events drive the TUI, event log, replay, and SDK callbacks via the `EventSink` protocol.

### Execution pipeline

1. `Runtime` (`runtime.py`) creates a `RunSession`, connects MCP apps via `MCPOrchestrator`, sets up event sinks (`CompositeEventSink`), and spawns the `Executor` as an async task.
2. `Executor` (`internal/executor.py`) drives step-by-step execution: builds prompts, calls the LLM in a loop, processes tool calls (control tools like `complete_step`/`request_clarification`, or app tools via MCP), handles approval gates, and emits events.
3. `LLMRuntime` (`llm/runtime.py`) wraps LiteLLM for model calls. Control tool schemas (`llm/tool_schemas.py`) are always injected alongside MCP tool schemas.
4. `MCPOrchestrator` (`mcp/orchestrator.py`) manages MCP app connections (stdio/HTTP), tool discovery, access policies (denylist + required_tools allowlist).
5. `HumanInterface` (`human.py`) is a Protocol with three methods: `request_approval`, `request_clarification`, `request_error_recovery`. Implementations: `TerminalHumanInterface` (Rich prompts), `AutoApproveHumanInterface` (testing), `ScriptedHumanInterface` (deterministic testing).

### Key internal modules

- `internal/executor.py` — The core loop. Has guard-rail constants: `MAX_TEXT_ONLY_ITERATIONS=5`, `MAX_TOOL_CALL_ITERATIONS=50`.
- `internal/context.py` — Message context trimming for LLM token budgets.
- `internal/tool_executor.py` — Bridges `MCPOrchestrator` results into runtime events/messages.
- `skills/parser.py` — Parses SKILL.md (YAML frontmatter + `### Step N: Title` headings + `[MARKER]` tags). Also contains `lint_skill()`.
- `skills/loader.py` — Resolves path (file or directory containing SKILL.md) and calls parser.
- `store/event_log.py` — JSONL event log writer + run directory manager (`.proceda/runs/<timestamp>_<short-id>/`).

### CLI (`cli/`)

Built with Typer. Commands: `run` (interactive terminal), `lint`, `replay`, `doctor`. Entry point: `proceda.cli.main:app`.

## Configuration

`proceda.yaml` (searched in `.` then `~/.config/proceda/`). Supports `${VAR}` and `${VAR:-default}` env expansion. Sections: `llm` (model, temperature, max_tokens), `apps` (MCP server configs), `dev`, `security` (tool_denylist), `logging` (run_dir, redact_secrets).

## Skill format

```markdown
---
name: my-workflow
description: What this workflow does
required_tools:
  - app__tool_name
---

### Step 1: Do something
Instructions for the LLM.

### Step 2: Validate
[APPROVAL REQUIRED]
Human must approve after this step completes.

### Step 3: Submit
[PRE-APPROVAL REQUIRED]
Human must approve before this step begins.
```

Steps must be sequential starting from 1. Markers: `[APPROVAL REQUIRED]` (post-step), `[PRE-APPROVAL REQUIRED]` (pre-step), `[OPTIONAL]`.

## Testing patterns

- Tests mirror source structure: `tests/test_skills/`, `tests/test_runtime/`, `tests/test_llm/`, etc.
- `pytest-asyncio` with `asyncio_mode = "auto"` — async test functions just work.
- Use `ScriptedHumanInterface` for deterministic approval/clarification sequences.
- Use `CollectorEventSink` (from `events.py`) to capture and assert on emitted events.
- Integration tests are marked with `@pytest.mark.integration`.

## Code style

- Python 3.11+. Ruff with line-length 100, rules: E, F, I, N, W, UP.
- Dataclasses for models (not Pydantic, despite it being a dependency — Pydantic is used only for config validation in some paths).
- `from __future__ import annotations` in all modules.
- All source files should have an ABOUTME comment as the module docstring.

---
> Source: [vivekhaldar/proceda](https://github.com/vivekhaldar/proceda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
