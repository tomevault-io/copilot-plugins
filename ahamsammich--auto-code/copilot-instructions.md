## auto-code

> `auto-code` is a Python package for running long-running autonomous coding sessions using the Claude Agent SDK and Google Agent Development Kit (ADK). It implements a two-agent pattern (Initializer + Coding Agent) to build complete applications from specifications or extend existing projects.

# auto-code Context for Gemini

## Project Overview

`auto-code` is a Python package for running long-running autonomous coding sessions using the Claude Agent SDK and Google Agent Development Kit (ADK). It implements a two-agent pattern (Initializer + Coding Agent) to build complete applications from specifications or extend existing projects.

- **Primary Goal:** Autonomous application generation and extension.
- **Modes:**
    - `greenfield`: Builds new applications from scratch (restricted to project dir).
    - `extend`: Adds features to existing projects (read access to entire project).
- **Providers:**
    - `claude` (Default): Uses Anthropic's Claude models.
    - `google`: Uses Google's Gemini models via `google-adk`.

## Environment & Setup

### Dependencies
- **Python:** 3.10+
- **Package Manager:** `pip` / `hatch`
- **External Tools:**
    - `ripgrep` (Required for Google provider's glob/grep tools).
    - `claude-code` CLI (Optional, for Claude provider).

### Authentication
- **Claude:** `ANTHROPIC_API_KEY` or `CLAUDE_CODE_OAUTH_TOKEN`.
- **Google:** `GOOGLE_API_KEY` (Required for Gemini/ADK support).

## Building and Running

### Installation
```bash
# Basic install
pip install -e .

# With Google ADK support (Gemini)
pip install -e ".[google]"

# With development dependencies
pip install -e ".[dev]"
```

### Execution
The entry point is the `auto-code` CLI command.

**Greenfield Mode (New App):**
```bash
auto-code --spec ./my_app_spec.txt
# With Gemini:
auto-code --spec ./my_app_spec.txt --provider google
```

**Extension Mode (Existing App):**
```bash
auto-code --extend ./features.txt
# With Gemini:
auto-code --extend ./features.txt --provider google
```

### Testing
Tests are located in `tests/` and use `pytest`.
```bash
pytest
```

### Linting & Formatting
Uses `ruff` for linting and `pyright` for type checking.
```bash
ruff check src/
pyright src/
```

## Development Conventions

### Code Style
- Follows standard Python conventions (PEP 8).
- Strictly typed (checked via `pyright`).
- Docstrings required for all public modules, classes, and functions.

### Architecture
- **`src/autonomous_coding/agent.py`**: Main orchestration loop.
- **`src/autonomous_coding/cli.py`**: CLI entry point and argument parsing.
- **`src/autonomous_coding/security.py`**: Defines the security model (Bash allowlist).
- **`src/autonomous_coding/providers/`**: Abstraction for LLM providers.
    - `claude.py`: Anthropic implementation.
    - `google.py`: Google ADK (Gemini) implementation.

### Security Model
The project employs a defense-in-depth strategy:
1.  **OS Sandbox:** Bash commands run in isolation.
2.  **Filesystem:** Restricted access based on mode (`greenfield` vs `extend`).
3.  **Allowlist:** Strict validation of executed commands (e.g., `git`, `npm`, `ls`, `cat`).
    - *Note:* Security logic is shared across providers via `security.py`.

## Gemini Provider Details
The Google provider (`src/autonomous_coding/providers/google.py`) integrates `google-adk`.
- **Default Model:** `gemini-3-flash-preview`.
- **Tools:** Implements built-in tools (read, write, edit, glob, grep, bash) and supports MCP.
- **Security:** Wraps the core `security.py` logic in an ADK-compatible `before_tool_callback`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AhamSammich)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AhamSammich)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
