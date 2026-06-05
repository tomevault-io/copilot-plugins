## rlmy

> Context for LLM agents working with this codebase. Read this to prevent recurring mistakes and understand non-obvious patterns.

# AGENTS.md

Context for LLM agents working with this codebase. Read this to prevent recurring mistakes and understand non-obvious patterns.

**Maintenance Principle**: This file should remain lean (few hundred lines). Only add lessons that caused actual user feedback and would waste time if repeated. See [`skills/agentsmd-update.md`](skills/agentsmd-update.md) for update guidelines. Use [`skills/review.md`](skills/review.md) before AND after any changes.

## What is rlmy

An interactive AI coding agent that runs in the terminal, built on DSPy's RLM (Recursive Language Model) framework. The LLM writes and executes Python code iteratively in a sandboxed REPL until it solves the problem. Connects to MCP servers for external tools (Slack, internal systems). Pronounced "ar-leh-mee."

## Build & Development Commands

This project uses `uv` for package management and has a virtual environment at `.venv/`.

```bash
# Create venv (if needed)
uv venv --python 3.12
uv sync

# Install dependencies (use uv, not bare pip)
uv add 'pillow>=10.0.0'

# Run the agent
python -m rlmy.cli

# Run tests (use venv python, not bare python)
.venv/bin/python -m pytest                    # all tests
.venv/bin/python -m pytest tests/test_rlmy_core.py  # single file
.venv/bin/python -m pytest -k "test_name"    # single test by name
```

## Architecture

The agent has a two-LM design:
- **Main LM** (strategist): drives the RLM reasoning loop — generates code, observes output, iterates
- **Sub LM** (worker): available to the agent's REPL code via `llm_query()` for subtasks

### Core Flow

`cli.py:main()` → config/wizard → sets env vars → `agent/main.py:run_agent()` → workspace selection → MCP connect → conversation loop with `InterruptableRLM`

## Common Patterns to Prevent Mistakes

### File Organization

**`src/rlmy/tools/`** — LLM-callable tools (shell, edit, MCP connectors)  
**`src/rlmy/agent/`** — Agent internals (RLM, UI utilities, commands, trajectory)

**Rule**: If it's called by the LLM as a tool → `tools/`. If it's UI/infrastructure for the agent itself → `agent/`.

**Example**: `clipboard.py` goes in `agent/` because it's a prompt_toolkit UI utility, not an LLM tool.

### Testing Philosophy

**Functional tests over exhaustive unit tests**. Test critical paths end-to-end, not isolated units.

- Create `tests/functional/` for workflow tests
- Mock at boundaries (e.g., `ImageGrab.grabclipboard`), not internal functions
- Focus on integration: "does the complete workflow work?"
- Avoid testing implementation details that may change
- Ask yourself "what's the 10% efforts that covers 90% of functional paths?"

**Example**: Test "clipboard → save → file exists → PIL reads" as one flow, not separate tests for each function.

### Virtual Environment

Always use the project's virtual environment:
- **Python**: `.venv/bin/python` (not bare `python` or `python3`)
- **Pytest**: `.venv/bin/python -m pytest` (not bare `pytest`)

The system Python may not have the right dependencies or may use a different package index.

### Dependency Management

- Add dependencies to `pyproject.toml` `dependencies = [...]` list
- Use `>=` for minimum versions (e.g., `"pillow>=10.0.0"`)
- Required dependencies fail fast at import time (not runtime)


## Documentation Standards for source code files

Source code is also living documentation. The code itself is already telling the "what", you need to intelligently write comments that tell the "why" (that which is not obvious).

### Required Headers

Python examples, but apply to source code in any language!

**File-level Header:**
Include at the top of files:
```python
"""
Purpose: [Main responsibility]
Usage: [Brief example]  
Key Components: [Main classes/functions]
Conventions: [Patterns other LLMs should follow]
"""
```

**Component-level Header:**
Include at the top of any significant component (classes, methods, functions...):
```python
"""
Purpose: [Class responsibility]
Attributes: [Key attributes]
Usage Patterns: [Conventions for other LLMs]
"""
```

### Documentation Rules for source code files
- **MUST** document complex chunks (files, classes, functions, code blocks) where **explanation reduces cognitive load**
- **MUST** document implicit context (non-obvious logic, project conventions, design decisions)
- **MUST** include examples for complex arguments
- **Focus on 'why'** not 'what'
- Update existing documentation when modifying code

---
> Source: [diego-lima/rlmy](https://github.com/diego-lima/rlmy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
