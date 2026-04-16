## google-adk-currency-agent

> This document provides foundational context, architectural overview, and development guidelines for the Currency Agent project.

# GEMINI.md - Currency Agent Project Context

This document provides foundational context, architectural overview, and development guidelines for the Currency Agent project.

## Project Overview

The **Currency Agent** is a reference implementation demonstrating the integration of three powerful AI agent technologies: **Agent Development Kit (ADK)**, **Model Context Protocol (MCP)**, and **Agent2Agent (A2A)**.

### Purpose
The project implements a specialized assistant for currency conversions. It uses an MCP server to provide real-time exchange rate tools (via the Frankfurter API) and an ADK agent to orchestrate the conversation, all while being served through an A2A-compatible interface.

### Key Technologies
- **Python (3.13+):** Core language.
- **Agent Development Kit (ADK):** Orchestration framework for creating the LLM agent (`google-adk`).
- **Model Context Protocol (MCP):** Standardizes how the agent connects to external tools and data sources (`fastmcp`).
- **Agent2Agent (A2A):** Protocol for enabling inter-agent communication and collaboration (`a2a-sdk`).
- **uv:** Ultra-fast Python package installer and resolver.
- **Taskfile:** Automation tool for common development workflows.
- **structlog:** Structured logging for Python.
- **Gemini:** The underlying LLM (e.g., `gemini-2.5-flash`).

### Architecture
1.  **MCP Server (`mcp-server/`):** Exposes various currency tools (e.g., `get_exchange_rate`, `convert_currency`, `get_rate_trend`, etc.) using `fastmcp`. It runs on port `8080`.
2.  **ADK Agent (`currency_agent/`):** A specialized LLM agent configured with a system instruction to handle only currency queries. It connects to the MCP server as a client.
3.  **A2A Server:** Wraps the ADK agent to make it A2A-compatible, running on port `10000`.
4.  **ADK Workbench:** (Optional) A local web interface to interact with and debug the agent, available via `task up:web` on port `8000`.

---

## Building and Running

The project uses `Task` for automation. Ensure `uv` and `go-task` are installed.

### Initialization
```bash
task init
```
This command syncs dependencies via `uv` and installs pre-commit hooks.

### Running the Services
To run the full stack, you need to start both servers in separate terminals:

1.  **Start MCP Server:**
    ```bash
    task up:mcp
    ```
2.  **Start A2A Agent Server:**
    ```bash
    task up:agent
    ```
3.  **Start ADK Workbench (Optional):**
    ```bash
    task up:web
    ```

### Testing
- **Run all tests:** `task test`
- **Run tests with coverage:** `task test:cov`
- **Run MCP tests:** `task test:mcp`
- **Run Agent tests:** `task test:agent`

### E2E Testing Workflow
To verify the full integration, follow these steps:
1.  **Kill existing servers:**
    ```bash
    task down
    ```
2.  **Start servers in background:**
    ```bash
    task up:mcp &
    task up:agent &
    ```
3.  **Run tests:**
    ```bash
    task test
    ```
4.  **Fix any identified issues.**
5.  **Run linting:**
    ```bash
    task lint
    ```
6.  **Shutdown background servers:**
    ```bash
    task down
    ```

### Linting and Formatting
```bash
task lint
```
Uses `ruff` for Python linting/formatting and `yamllint` for YAML files.

---

## Development Conventions

### Package Management
- Always use `uv` for managing dependencies.
- The project has a root `pyproject.toml` and a separate one in `mcp-server/`.
- `uv.lock` files are present and should be committed.

### Coding Standards
- **Linting:** `ruff` is the primary linter and formatter.
- **Pre-commit Hooks:** Enforced via `.pre-commit-config.yaml`. Hooks include trailing whitespace removal, YAML validation, `ruff`, and `gitleaks`.
- **Environment Variables:** Managed via `.env` file. A `.env.example` is provided as a template. Key variables:
    - `GOOGLE_API_KEY`: For direct Gemini API access.
    - `GOOGLE_CLOUD_PROJECT`, `GOOGLE_CLOUD_LOCATION`: For Vertex AI access.
    - `MCP_SERVER_URL`: URL where the agent can find the MCP server.
    - `LOG_JSON`: Set to `TRUE` to enable JSON structured logging (default is human-readable console output).

### Contribution Workflow
1.  Ensure all tests pass before submitting changes (`task test`).
2.  Run `task lint` to ensure code style compliance.
3.  Pre-commit hooks will automatically run on commit to catch common issues.

### Project Structure
- `currency_agent/`: Contains the main ADK agent logic and A2A wrapper.
- `mcp-server/`: Contains the MCP tool server implementation.
- `images/`: Documentation assets.
- `Taskfile.yml`: Automation task definitions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nhsy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
