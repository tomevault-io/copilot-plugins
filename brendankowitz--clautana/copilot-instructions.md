## clautana

> **Clautana** (Claude + Cortana) is a VS Code extension that implements an AI-powered multi-agent orchestration system. It enables a primary "Orchestrator" agent to break down complex coding tasks, manage them via a Kanban board, and spawn specialist "Worker" agents to execute them.

# Clautana Project Context

## Project Overview

**Clautana** (Claude + Cortana) is a VS Code extension that implements an AI-powered multi-agent orchestration system. It enables a primary "Orchestrator" agent to break down complex coding tasks, manage them via a Kanban board, and spawn specialist "Worker" agents to execute them.

### Core Architecture

*   **Orchestrator Agent (`OrchestratorAgent.ts`):** The central "brain" of the system. It receives user input, plans work, creates User Stories, and manages the lifecycle of worker agents. It uses the Model Context Protocol (MCP) to interact with the system.
*   **Worker Agents:** Specialized agents spawned by the Orchestrator to perform specific tasks (e.g., "Backend Engineer", "Code Reviewer"). They operate independently but coordinate through the Orchestrator.
*   **Agent Pool (`AgentPool.ts`):** Manages the execution environment for worker agents.
*   **MCP (Model Context Protocol):** The communication standard used for agents to access tools (file system, memory, git, etc.).
    *   `McpManager.ts`: Manages MCP server lifecycles (stdio/http transports).
    *   `MailMcpServer.ts`: Handles inter-agent messaging.
    *   `MemoryMcpServer.ts`: Provides persistent memory (facts, lessons, playbooks).
*   **Kanban System:** Work is tracked as "User Stories" stored in `.clautana/workitems/`. This file-based system serves as the source of truth for task status.
*   **Frontend:** A React-based webview (`webview-ui`) displays the chat interface, agent status, and Kanban board.

## Key Directories

*   `src/multi-agent-harness/`: The main extension source code.
    *   `src/coordinator/`: Logic for the Orchestrator and Agent Pool.
    *   `src/mcp/`: Implementations of MCP servers (Mail, Memory, etc.).
    *   `src/kanban/`: Logic for managing work items and the file-based board.
    *   `src/providers/`: VS Code UI providers (Tree Views, Webviews).
*   `src/multi-agent-harness/webview-ui/`: The React frontend for the extension's UI.
*   `docs/`: Project documentation and architectural decision records (ADRs).
*   `.clautana/`: Runtime directory for agent state, memory, and work items.

## Development & Build

### Prerequisites
*   Node.js & npm
*   VS Code

### Build Commands
Run these from `src/multi-agent-harness/`:

*   **Build All:** `npm run build` (Builds both extension and webview)
*   **Build Extension:** `npm run build:extension` (Webpack)
*   **Build Webview:** `npm run build:webview` (Vite)
*   **Watch Mode:** `npm run watch`

### Running the Extension
1.  Open the project in VS Code.
2.  Press `F5` to launch the "Extension Host" window.
3.  Use the command palette (`Ctrl+Shift+P`) to access Clautana commands (e.g., `Clautana: Open Clautana Panel`).

## Orchestrator Logic & Workflow

The Orchestrator follows a strict workflow defined in `OrchestratorAgent.ts`:

1.  **Receive Task:** User submits a request via the chat panel.
2.  **Plan:** Orchestrator analyzes the request and breaks it down.
3.  **Create Stories:** Uses `create_workitem` to create Kanban cards (User Stories) for distinct units of work.
    *   *Note:* Features are often referenced from `docs/features/`.
4.  **Spawn Agents:** Uses `spawn_agent` to create workers, assigning them specific User Stories via `workItemId`.
5.  **Monitor:** Orchestrator monitors progress, receiving messages from agents via `inbox`.
6.  **Review:** Completed work is reviewed (often by a specialized "Reviewer" agent).
7.  **Complete:** Orchestrator reports final success to the user.

## Conventions

*   **MCP First:** Agents should rely on MCP tools for all interactions with the OS and IDE.
*   **File-Based State:** The `.clautana` directory is used for persistence. Do not rely on in-memory state for long-term data.
*   **React UI:** The UI is built with React and Vite, communicating with the extension backend via the VS Code Webview API.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brendankowitz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
