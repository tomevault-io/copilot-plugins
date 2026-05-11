## devduck

> - **What:** Self-modifying AI agent that hot-reloads its own code

# AGENTS.md — DevDuck

## Project: DevDuck
- **What:** Self-modifying AI agent that hot-reloads its own code
- **Language:** Python 3.10–3.13
- **Framework:** [Strands Agents SDK](https://strandsagents.com)
- **Entry point:** `devduck/__init__.py` (single-file core)
- **Package:** `pip install devduck` / `pipx install devduck`
- **License:** Apache 2.0

## Architecture

DevDuck is a single `DevDuck` class in `__init__.py` that auto-initializes on import.
The module itself is callable: `import devduck; devduck("query")`.

```
devduck/
├── __init__.py              # Core agent: DevDuck class, REPL, CLI, session recording, ambient mode
├── tui.py                   # Multi-conversation Textual TUI (concurrent panels, streaming markdown)
├── landing.py               # Rich landing screen for REPL mode
├── callback_handler.py      # Streaming callback handler for CLI
├── asciinema_callback_handler.py  # Callback handler that records .cast files
├── agentcore_handler.py     # HTTP handler for Bedrock AgentCore deployment
├── tools/                   # 60+ built-in tools
│   ├── system_prompt.py     # Self-improvement via prompt management
│   ├── manage_tools.py      # Runtime tool add/remove/create/fetch
│   ├── manage_messages.py   # Conversation history management
│   ├── websocket.py         # WebSocket server (per-message agent)
│   ├── zenoh_peer.py        # P2P auto-discovery networking
│   ├── agentcore_proxy.py   # Unified mesh relay (Zenoh + AgentCore + browser)
│   ├── unified_mesh.py      # Ring context shared memory
│   ├── mesh_registry.py     # File-based agent discovery with TTL
│   ├── tasks.py             # Background parallel agent tasks
│   ├── scheduler.py         # Cron and one-time job scheduling
│   ├── telegram.py          # Telegram bot integration
│   ├── slack.py             # Slack integration
│   ├── whatsapp.py          # WhatsApp via wacli
│   ├── tui.py               # Agent-side TUI content push
│   ├── speech_to_speech.py  # Real-time voice (Nova Sonic, OpenAI, Gemini)
│   ├── lsp.py               # Language Server Protocol diagnostics
│   ├── use_mac.py           # Unified macOS control
│   ├── apple_notes.py       # Apple Notes management
│   ├── use_spotify.py       # Spotify control
│   └── ...                  # tcp, ipc, mcp_server, scraper, rl, etc.
└── tools/ (hot-reload)      # ./tools/*.py auto-loaded at runtime
```

## Key Design Patterns

### Self-Awareness
The system prompt includes the agent's **complete source code** via `get_own_source_code()`. This means I can inspect my own implementation to answer questions accurately rather than relying on potentially stale conversation context.

### Self-Healing
`_self_heal(error)` retries initialization on failure (max 2 attempts). Context window overflow is auto-detected — history is cleared and the query is retried.

### Hot-Reload
A background `_file_watcher_thread` monitors `__init__.py` for changes. On detection, `os.execv()` restarts the process. If the agent is executing, reload is deferred until completion (`_reload_pending`).

### Dynamic Tool Loading
Tools are configured via `DEVDUCK_TOOLS` env var in `package:tool1,tool2;package2:tool3` format. Additional tools can be loaded at runtime via `manage_tools(action="add", ...)` or by dropping `.py` files in `./tools/`.

### Multi-Protocol Servers
On startup, DevDuck auto-starts enabled servers (WebSocket on 10001, Zenoh P2P, Mesh Relay on 10000). Port conflicts are detected and the next available port is used.

### AGENTS.md Auto-Load
This file is automatically read from the working directory and injected into the system prompt on startup — see `_build_system_prompt()` in `__init__.py`.

## Conventions

### Response Style
- **Minimal words** — brief, direct responses
- **Maximum parallelism** — independent tool calls are batched
- **Speed is paramount** — efficiency over verbosity

### Tool Calls
- Use `shell` for system commands
- Use `editor` for file modifications (str_replace, create, insert)
- Use `file_read` for reading files
- Use `file_write` for writing files
- Use `use_github` for GitHub GraphQL API
- Use `use_agent` for spawning sub-agents with different models

### Error Handling
- Self-healing on initialization errors
- Auto context window recovery (clear history + retry)
- Graceful degradation to "minimal mode" if all recovery fails

### State Management
- Conversation history in `agent.messages`
- Shell history in `~/.devduck_history`
- Logs in `/tmp/devduck/logs/devduck.log`
- Session recordings in `/tmp/devduck/recordings/`
- Scheduler jobs persisted to disk
- SQLite memory at default path

## Access Methods

| Protocol | Port | Description |
|----------|------|-------------|
| CLI/REPL | — | `devduck` interactive mode |
| TUI | — | `devduck --tui` multi-conversation Textual UI |
| MCP stdio | — | `devduck --mcp` for Claude Desktop |
| Mesh Relay | 10000 | Browser peers + AgentCore agents |
| WebSocket | 10001 | Per-message streaming agent |
| TCP | 10002 | Raw socket (opt-in) |
| MCP HTTP | 10003 | Model Context Protocol (opt-in) |
| Zenoh P2P | multicast | Auto-discovery across terminals/networks |

## Model Selection

Auto-detects in priority order:
Bedrock → Anthropic → OpenAI → GitHub → Gemini → Cohere → Writer → Mistral → LiteLLM → LlamaAPI → SageMaker → LlamaCpp → MLX → Ollama

Override with `MODEL_PROVIDER` and `STRANDS_MODEL_ID` env vars.

## Testing & Development

```bash
git clone git@github.com:cagataycali/devduck.git
cd devduck
python3.13 -m venv .venv && source .venv/bin/activate
pip install -e .
devduck
```

## Important Notes for AI Agents

1. **Source code is truth** — always check `__init__.py` over conversation memory
2. **Hot-reload is active** — editing source files triggers automatic process restart
3. **AGENTS.md is auto-loaded** — changes here affect the next agent initialization
4. **Context overflow is handled** — long sessions auto-recover by clearing history
5. **Tool consent is bypassed** — `BYPASS_TOOL_CONSENT=true` is set by default
6. **Ambient mode** — background thinking can be active; check `ambient` status
7. **Session recording** — may be capturing all tool calls and messages
8. **Mesh context** — ring context from browser/cloud agents may be injected into queries

---
> Source: [cagataycali/devduck](https://github.com/cagataycali/devduck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
