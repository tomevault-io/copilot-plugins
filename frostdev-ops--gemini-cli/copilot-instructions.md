## 05-daemons-ipc

> Use this rule to understand the roles of the HAPPE/IDA/mcp-hostd daemons, how HAPPE and IDA communicate via IPC, find the IPC message structure definitions, trace the memory retrieval flow, and identify daemon responsibilities.

# Gemini Rust Suite: Daemons and IPC

For more complex, persistent scenarios, the system uses background daemons communicating via Inter-Process Communication (IPC).

*   **`happe` Daemon (@happe/README.md)**: The Host Application Environment. Orchestrates user interactions, LLM calls, MCP tool execution, and communication with `IDA`.
*   **`ida` Daemon (@ida/README.md)**: Internal Dialogue App. Manages persistent memory interactions (retrieval/storage via Memory MCP Server) and other background cognitive tasks.
*   **`mcp-hostd` Daemon ([mcp/README.md](mdc:mcp/README.md))**: Standalone MCP host process. Manages connections to MCP tool servers.
*   **IPC Definitions (`gemini-ipc`)**: The `gemini-ipc` crate ([ipc/README.md](mdc:ipc/README.md)) centralizes message structures for communication between these components.
    *   `internal_messages.rs` ([ipc/src/internal_messages.rs](mdc:ipc/src/internal_messages.rs)): Defines messages between `HAPPE` and `IDA`.
    *   `daemon_messages.rs` ([ipc/src/daemon_messages.rs](mdc:ipc/src/daemon_messages.rs)): Defines messages between clients (like `gemini-cli`) and `mcp-hostd`.
*   **Communication Flow:**
    *   `User/Client` -> `HAPPE`
    *   `HAPPE` <-> `IDA` (via IPC, using `internal_messages`)
    *   `IDA` -> `Memory MCP Server` (via MCP)
    *   `HAPPE` -> `Main LLM`
    *   `HAPPE` -> `Other MCP Servers` (via MCP, potentially through `mcp-hostd`)
    *   `gemini-cli` -> `mcp-hostd` (optional, via IPC, using `daemon_messages`)

---
> Source: [frostdev-ops/gemini-cli](https://github.com/frostdev-ops/gemini-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
