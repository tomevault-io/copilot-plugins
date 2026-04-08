## agents-hub

> This project is a Go-based implementation of an Agent-to-Agent (A2A) Local Hub. It serves as a central orchestrator that allows various CLI-based AI agents (like Claude Code, Gemini CLI, Codex, and Vibe) to communicate and collaborate.

# A2A Local Hub (Go)

This project is a Go-based implementation of an Agent-to-Agent (A2A) Local Hub. It serves as a central orchestrator that allows various CLI-based AI agents (like Claude Code, Gemini CLI, Codex, and Vibe) to communicate and collaborate.

## Project Overview

The hub exposes a JSON-RPC 2.0 API over both Unix domain sockets and HTTP. It includes a Terminal User Interface (TUI) for interactive management and messaging.

### Core Features

*   **Multi-Agent Orchestration:** Manages multiple AI agents, allowing them to communicate and perform tasks.
*   **Transport Layer:** Supports JSON-RPC 2.0 over Unix domain sockets (NDJSON) and HTTP.
*   **Terminal UI:** An interactive TUI built with Bubble Tea for monitoring status, managing agents, and sending messages.
*   **Extensibility:** Wraps external CLI tools as agents using a generic `CLIAgent` wrapper.
*   **Persistence:** Locally persists tasks, contexts, and settings.

### Architecture

The project follows a standard Go project structure:

*   `cmd/agents-hub/`: Contains the main entry point (`main.go`).
*   `internal/`: Contains the core application logic.
    *   `agents/`: Definitions for different agents and the generic `CLIAgent` wrapper.
    *   `hub/`: Core server logic, agent registry, task and context management.
    *   `transport/`: Implementation of Unix socket and HTTP transports.
    *   `jsonrpc/`: JSON-RPC 2.0 request handling and type definitions.
    *   `tui/`: Bubble Tea-based terminal user interface.
    *   `types/`: Shared type definitions (A2A protocol, messages, tasks).
    *   `utils/`: Utility functions (logging, ID generation).
*   `SPECIFICATION.md`: Defines the A2A protocol and system architecture.

## Building and Running

### Prerequisites

*   Go 1.24.0 or later.
*   External agent CLIs (e.g., `claude`, `gemini`, `codex`, `vibe`) must be installed and in your `PATH` if you intend to use them.

### Build

To build the project:

```bash
go build ./cmd/agents-hub
```

### Run

To run the hub (defaults to launching the TUI):

```bash
./agents-hub
```

To start the hub server in the foreground (without TUI):

```bash
./agents-hub start --foreground
```

To send a message via CLI:

```bash
./agents-hub send codex "Write a hello world function in Go"
```

## Development Conventions

*   **Language:** Go (1.24).
*   **Architecture:** Modular design with separation of concerns (transport, logic, agents).
*   **Communication:** Strictly follows the JSON-RPC 2.0 specification as defined in `SPECIFICATION.md`.
*   **UI:** Uses `charmbracelet/bubbletea` for the TUI.
*   **Error Handling:** Standard Go error handling, mapping to JSON-RPC error codes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/azs06)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/azs06)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
