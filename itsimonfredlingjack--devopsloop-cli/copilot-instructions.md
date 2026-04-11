## devopsloop-cli

> **vibe-cli** is an Agentic AI coding assistant with a terminal-based user interface (TUI) built using Textual. It empowers developers to interact with LLMs directly from their terminal to perform coding tasks, run commands, and manage their workspace with built-in security controls.

# vibe-cli

**vibe-cli** is an Agentic AI coding assistant with a terminal-based user interface (TUI) built using Textual. It empowers developers to interact with LLMs directly from their terminal to perform coding tasks, run commands, and manage their workspace with built-in security controls.

## Project Overview

*   **Type:** Python CLI Application
*   **Core Frameworks:** Textual (UI), Rich (Formatting), Pydantic (Data Validation), HTTPX (Async Networking).
*   **Goal:** Provide a secure, extensible, and interactive terminal environment for AI-assisted development.

## Architecture

The project is structured as follows:

*   **`vibe_cli/`**: The main package directory.
    *   **`__main__.py`**: Entry point for the application.
    *   **`config.py`**: Handles configuration loading from `~/.config/vibe/config.toml`.
    *   **`agent/`**: Core agent logic.
        *   `loop.py`: The main loop (LLM -> Tool Execution -> Context Update).
        *   `context.py`: Manages conversation history and token compression.
        *   `checkpoint.py`: Handles saving and restoring conversation states.
    *   **`models/`**: Pydantic models for data structures.
        *   `messages.py`: Defines `Message`, `ToolCall`, `ToolResult`.
        *   `tools.py`: Defines tool schemas (`ToolDefinition`).
    *   **`providers/`**: LLM backend abstractions.
        *   `base.py`: Abstract base class for providers.
        *   `openai_compat.py`: Generic OpenAI-compatible client (works with OpenAI, Ollama, etc.).
    *   **`tools/`**: Built-in tool implementations.
        *   `base.py`: Tool registry and plugin loading logic.
        *   `filesystem.py`: File operations (Read, Write, Replace) with path traversal protection.
        *   `shell.py`: Command execution with allowlist/blocklist security.
        *   `git.py`: Git integration.
    *   **`ui/`**: Textual-based user interface.
        *   `app.py`: Main application class `VibeApp`.
        *   `widgets.py`: Custom widgets like Chat view, Input area.
        *   `theme.py`: UI styling and ASCII art.

## Setup and Usage

### Installation

1.  **Create a virtual environment:**
    ```bash
    python3 -m venv venv
    source venv/bin/activate
    ```
2.  **Install dependencies (editable mode):**
    ```bash
    pip install -e ".[dev]"
    ```

### Running the Application

To start the Vibe CLI:
```bash
vibe
```

### Configuration

Configuration is stored in `~/.config/vibe/config.toml`.

Example `config.toml`:
```toml
default_provider = "ollama"

[providers.ollama]
type = "ollama"
model = "granite3.1-dense:2b"
base_url = "http://localhost:11434"
auto_switch = true
```

## Development Workflow

### Testing

Run tests using `pytest`:

```bash
# Run all tests
pytest -xvs

# Run specific test file
pytest tests/test_models.py -xvs
```

### Linting and Formatting

The project uses `ruff`:

```bash
# Lint
ruff check --fix .

# Format
ruff format .
```

## Security Model

*   **Workspace Confinement:** Filesystem tools are restricted to the workspace directory to prevent path traversal.
*   **Shell Command Filtering:**
    *   **Allowlist:** `ls`, `cat`, `grep`, `git`, `pytest`, `npm`, etc.
    *   **Blocklist:** `rm -rf`, `> /dev/`, `sudo`.
*   **Dangerous Tool Confirmation:** Tools like `write_file` or `run_command` can be configured to require user confirmation.
*   **Plugin Security:** Workspace plugins (`.vibe/tools/*.py`) are disabled by default. Enable with `VIBE_ALLOW_WORKSPACE_PLUGINS=1`.

## Key Files for Context

*   **`vibe_cli/tools/base.py`**: Understanding how tools are registered and executed.
*   **`vibe_cli/agent/loop.py`**: The "brain" of the agent, handling the reasoning loop.
*   **`vibe_cli/ui/app.py`**: The entry point for the TUI, managing application lifecycle.
*   **`CLAUDE.md`**: Contains detailed architectural notes and context for AI assistants.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itsimonfredlingjack)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/itsimonfredlingjack)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
