## agent-studio

> This document provides a comprehensive overview of the `agentx` project, its purpose, architecture, and instructions for building and running it.

# GEMINI.md

This document provides a comprehensive overview of the `agentx` project, its purpose, architecture, and instructions for building and running it.

## Project Overview

`agentx` is a desktop application written in Rust using the `gpui` framework. It serves as a demonstration and development environment called "Agent Studio," designed for interacting with an AI agent.

The application provides a multi-panel, dock-based workspace that includes:
*   **Conversation Panel:** Displays the turn-by-turn chat history between the user and the agent, including text, code snippets, and agent tool usage.
*   **Task List Panel:** Shows a to-do list or plan that the agent is executing.
*   **Code Editor Panel:** A built-in editor for viewing and modifying code.
*   **Chat Input Panel:** The input area for the user to type messages to the agent.
*   **Welcome Panel:** An initial screen for the application.

The project heavily utilizes the `gpui-component` library, building a modular UI where each panel is a distinct component. The current implementation uses hard-coded mock data to render a sample conversation, showcasing the various UI components and their intended interactions.

## Architecture

*   **Core Framework:** Rust with `gpui` for the GUI.
*   **Main Entry Point:** `src/main.rs` initializes the application, sets up the main window, and creates the `DockWorkspace`.
*   **Library Crate:** `src/lib.rs` defines the primary module structure, exports shared components, and contains the core `DockPanel` trait.
*   **Component System:** The UI is built around a `DockPanel` trait (`src/lib.rs`). Each major UI element (e.g., `ConversationPanel`, `ListTaskPanel`) implements this trait to integrate into the main dockable workspace.
*   **UI Components:** The building blocks of the UI, such as `UserMessage`, `AgentMessage`, `ToolCallItem`, and `AgentTodoList`, are defined in `src/components/`. These are used to render the conversation flow.
*   **State & Layout:** The application saves its dock layout state to a `docks-agentx.json` file, allowing the UI configuration to persist between sessions.

## Building and Running

This is a standard Rust project managed with Cargo.

### Build
To build the project, run:
```bash
cargo build
```

### Run
To run the application, use:
```bash
cargo run
```

### Test
To run tests, use:
```bash
cargo test
```

## Development Conventions

*   **Component-Based:** The UI is broken down into reusable components and panels. New functionality should be encapsulated within a new struct that implements the `DockPanel` trait.
*   **Styling:** Styling is done in code using the methods provided by `gpui` and the active theme.
*   **Mock Data:** The `ConversationPanel` (`src/conversation.rs`) currently uses extensive hard-coded data for UI development and demonstration. This is a useful pattern for developing UI components independently of backend logic.
*   **Dependencies:** Project dependencies are managed in `Cargo.toml`. Key dependencies include `gpui`, `gpui-component`, and `serde`.

---
> Source: [sxhxliang/agent-studio](https://github.com/sxhxliang/agent-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
