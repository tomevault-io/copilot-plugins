## 03-mcp

> Use this rule to learn how external tools are integrated via MCP, find the MCP host code and server configuration file, identify built-in and external (E5 embedding) servers, and trace how Gemini function calls are executed as MCP tool requests.

# Gemini Rust Suite: Model Context Protocol (MCP)

MCP allows Gemini to use external tools and services. This is primarily handled by the `gemini-mcp` crate ([mcp/README.md](mdc:mcp/README.md)).

## Key Components:

*   **MCP Host:** The core logic resides in `McpHost` ([mcp/src/host.rs](mdc:mcp/src/host.rs)). It discovers, manages, and communicates with MCP servers.
    *   Can run embedded within the CLI.
    *   Can run as a standalone daemon: `mcp-hostd` ([mcp/src/bin/mcp-hostd.rs](mdc:mcp/src/bin/mcp-hostd.rs)).
*   **MCP Servers:** External processes providing tools (capabilities). The host connects to them based on configuration.
    *   **Configuration:** Defined in `~/.config/gemini-suite/mcp_servers.json`. Specifies server name, transport (stdio, sse, websocket), command to launch (for stdio), etc. See [mcp/README.md](mdc:mcp/README.md) for format.
    *   **Transports:** Supports `Stdio`, `SSE`, and `WebSocket`.
    *   **Communication:** Uses JSON-RPC 2.0.
*   **Built-in Servers:** The source code for common servers is included within `gemini-mcp`:
    *   `command` ([mcp/src/servers/command/mod.rs](mdc:mcp/src/servers/command/mod.rs)): Executes shell commands (requires user confirmation by default).
    *   `filesystem` ([mcp/src/servers/filesystem/mod.rs](mdc:mcp/src/servers/filesystem/mod.rs)): Reads/writes files, lists directories.
    *   `memory_store` ([mcp/src/servers/memory_store/mod.rs](mdc:mcp/src/servers/memory_store/mod.rs)): Simple key-value storage (distinct from the main `gemini-memory` LanceDB store). *Note: This server might be less relevant now with the dedicated E5 embedding server.* 
    *   These can be run via CLI flags (e.g., `gemini --filesystem-mcp`) or configured in `mcp_servers.json` to be launched by the host.
*   **External Python E5 Embedding Server:** A crucial server for semantic memory.
    *   Located in [mcp_embedding_server/](mdc:mcp_embedding_server).
    *   Provides the `embed` tool using E5 models.
    *   Communicates via `stdio` JSON-RPC.
    *   See [mcp_embedding_server/README.md](mdc:mcp_embedding_server/README.md) and the server script [mcp_embedding_server/server.py](mdc:mcp_embedding_server/server.py).
*   **Gemini Integration:**
    *   The MCP host translates server capabilities into Gemini `FunctionDeclaration`s.
    *   It dispatches Gemini `FunctionCall`s to the appropriate MCP server's tool.
    *   Handles user confirmation for potentially sensitive tools.

---
> Source: [frostdev-ops/gemini-cli](https://github.com/frostdev-ops/gemini-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
