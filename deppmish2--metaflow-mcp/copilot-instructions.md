## metaflow-mcp

> This project, `outerbounds-mcp`, is a Python-based tool that provides a command-line interface to interact with the Metaflow Client API. It leverages the `fastmcp` library to create a server that exposes several tools for inspecting Metaflow flows, runs, artifacts, and logs. This allows users to debug and understand their Metaflow executions from the terminal.

# Project Overview

This project, `outerbounds-mcp`, is a Python-based tool that provides a command-line interface to interact with the Metaflow Client API. It leverages the `fastmcp` library to create a server that exposes several tools for inspecting Metaflow flows, runs, artifacts, and logs. This allows users to debug and understand their Metaflow executions from the terminal.

# Building and Running

## Prerequisites

*   Python 3.12
*   uv

## Installation

1.  **Sync the virtual environment:**
    ```bash
    uv sync
    ```

## Running the Server

To start the `metaflow-inspector` server, run the following command:

```bash
metaflow-inspector
```

This will start a local server that MCP clients can connect to.

# Development Workflow

This project follows a specific iterative development loop when modifying the MCP server.

1.  **Modify Code:** Make changes to the Python source code, primarily `metaflow_inspector.py` to add or modify tools. If dependencies are changed, update `pyproject.toml`.
2.  **Sync Environment:** If `pyproject.toml` was modified, run `uv sync` to update the environment.
3.  **Restart Server:** The user must manually stop and restart the `metaflow-inspector` server process for any code changes to take effect. They do this by running `/mcp refresh` in the Gemini CLI
4.  **Test:** The AI agent will execute the modified or newly added tools to verify the changes are working as expected.

# Development Conventions

The project follows standard Python development conventions. It uses `pyproject.toml` for dependency management and project metadata. The main application logic is contained in `metaflow_inspector.py`.

# Available Tools

The following tools are available through the `metaflow-inspector` server:

*   **list_flows**: Returns a list of all unique flow names that have been run.
*   **list_runs**: Lists the most recent runs for a given flow.
*   **get_latest_successful_run**: Finds the latest successfully completed run for a given flow.
*   **get_run_artifacts**: Retrieves data artifacts from a specified run. If a list of artifact names is provided, it fetches and pretty-prints the requested artifacts.
*   **get_step_logs**: Fetches the combined stdout and stderr logs for a specific step within a run.
*   **list_run_steps**: Lists the names of all steps for a given run.
*   **get_run_logs**: Fetches the combined stdout and stderr logs for all steps in a given run.
*   **list_runs_by_tag**: Finds all runs for a given flow that have a specific tag or tags.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deppmish2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/deppmish2)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
