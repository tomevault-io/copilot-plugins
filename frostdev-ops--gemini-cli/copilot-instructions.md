## 01-project-overview

> Use this rule to get a high-level project overview, identify the core function of each crate, understand major subsystems, and find the entry-point documentation for components.

# Gemini Rust Suite: Project Overview and Core Crates

This project is a Cargo workspace providing a suite of Rust crates for interacting with Google Gemini models, featuring tool usage (MCP), persistent memory, and a CLI.

See the main [README.md](mdc:README.md) for a full overview.

## Core Crates:

*   **`gemini-core` ([core/README.md](mdc:core/README.md))**: Foundational components. Includes the async `GeminiClient`, API types, configuration (`GeminiConfig`), error handling, and shared JSON-RPC types.
*   **`gemini-ipc` ([ipc/README.md](mdc:ipc/README.md))**: Centralizes Inter-Process Communication message definitions (structs/enums) used between daemons (`HAPPE`, `IDA`, `mcp-hostd`) and clients (`gemini-cli`).
*   **`gemini-mcp` ([mcp/README.md](mdc:mcp/README.md))**: Implements the Model Context Protocol (MCP) **host** side. Manages discovering, launching, and communicating with MCP **servers** (tools). Includes the `mcp-hostd` binary and built-in server implementations.
*   **`gemini-memory` ([memory/README.md](mdc:memory/README.md))**: Implements persistent semantic memory using LanceDB. Relies on an MCP host (`McpHostInterface`) for embedding generation.
*   **`gemini-cli` ([cli/README.md](mdc:cli/README.md))**: The main command-line interface. Integrates the other crates for user interaction, tool use, and memory.
*   **`HAPPE` (@happe/README.md)**: Host Application Environment daemon. Manages interactions between user, LLM, `IDA`, and MCP servers. Intended as the primary execution environment.
*   **`IDA` (@ida/README.md)**: Internal Dialogue App daemon. Manages persistent memory and background cognitive tasks, communicating with `HAPPE` via IPC.

---
> Source: [frostdev-ops/gemini-cli](https://github.com/frostdev-ops/gemini-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
