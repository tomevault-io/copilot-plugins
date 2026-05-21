## 02-cli

> Use this rule to understand CLI configuration (API key, model, prompts, history/memory flags), history persistence via the wrapper function, interaction modes, MCP/memory integration points, and where to configure tool servers.

# Gemini Rust Suite: Command Line Interface (CLI)

The primary user interface is provided by the `gemini-cli` crate ([cli/README.md](mdc:cli/README.md)).

## Key Components:

*   **Binary:** `gemini-cli-bin` is the compiled executable.
*   **Wrapper:** The `gemini` command (usually installed via `install.sh` to `~/.bashrc` or `~/.zshrc`) is a crucial wrapper around `gemini-cli-bin` that manages session environment variables (`GEMINI_SESSION_ID`) for history persistence across separate command invocations. See the wrapper function definition in [README.md](mdc:README.md).
*   **Dependencies:** Leverages `gemini-core` for API communication, `gemini-mcp` for tool integration, and `gemini-memory` for memory features.
*   **Configuration:**
    *   `~/.config/gemini-suite/config.toml`: Stores API key, default model, system prompt, feature flags (history, memory broker, auto memory). Managed via `gemini --set-...` flags or manual editing. See [cli/README.md](mdc:cli/README.md).
    *   `~/.config/gemini-suite/mcp_servers.json`: Configures connections to MCP servers (used by the embedded MCP host or the `mcp-hostd` daemon). See [mcp/README.md](mdc:mcp/README.md).
*   **Interaction Modes:** Supports single-shot prompts (default), interactive chat (`-i`), task loops (`-t`), and combined interactive task mode (`-i -t`). These are detailed in [INTERACTIVE-TASKS.md](mdc:INTERACTIVE-TASKS.md).
*   **History:** Saved in `~/.local/share/gemini-suite/history/`. Requires the wrapper function for history across commands.
*   **Built-in Server Execution:** Can run built-in MCP servers directly via flags (`--filesystem-mcp`, `--command-mcp`, `--memory-store-mcp`).

---
> Source: [frostdev-ops/gemini-cli](https://github.com/frostdev-ops/gemini-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
