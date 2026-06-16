## switchplane

> Switchplane is a Python runtime control plane for agent-based task execution. It is LangGraph-native — tasks are defined as LangGraph StateGraph graphs. Each app built with Switchplane becomes a standalone CLI with its own isolated daemon and runtime directory.

# CLAUDE.md — Switchplane

## What is this project?

Switchplane is a Python runtime control plane for agent-based task execution. It is LangGraph-native — tasks are defined as LangGraph StateGraph graphs. Each app built with Switchplane becomes a standalone CLI with its own isolated daemon and runtime directory.

## Architecture in one sentence

Each app gets its own CLI that sends requests over a Unix socket to a per-app daemonized control plane, which spawns agent subprocesses that communicate bidirectionally over a per-agent Unix socketpair using length-prefixed JSON framing.

## Project layout

```
src/switchplane/           # Main package (pip-installable)
  __init__.py              # Public API re-exports: Application, Shell, Task, command, Field
  app.py                   # Application container and McpServerConfig
  shell.py                 # Sandboxed subprocess execution with path/command allowlists, LangChain tool factory
  agent.py                 # AgentSpec and AgentRecord models
  task.py                  # Task ABC, TaskRecord, TaskStatus, @command decorator, parameter introspection
  protocol.py              # IPC message types: CliRequest/Response, AgentEvent/Command
  transport.py             # Unix socket server (async) + client (sync), 4-byte length-prefixed framing
  persistence.py           # SQLite Store (aiosqlite, WAL mode) — tasks, agents, events tables
  checkpoint.py            # LangGraph BaseCheckpointSaver backed by SQLite
  discovery.py             # Agent/task discovery via module import (no entry point discovery)
  daemon.py                # Daemonization (double-fork), signal handling, idle timer, RuntimePaths
  control_plane.py         # Central orchestrator — request dispatch, single app, idle shutdown
  subprocess_manager.py    # Agent subprocess lifecycle — socketpair creation, event reading, command sending
  agent_runtime.py         # Runs inside agent subprocess — AgentContext, bidirectional IPC over socketpair
  cli.py                   # Click CLI factory — build_cli(app) generates run, runtime, agent, task command groups
  tui.py                   # prompt_toolkit full-screen TUI — tab-based task viewer with system tab [0]
  fmt.py                   # Shared formatting utilities (e.g. tree rendering for detail payloads)
  config.py                # TOML config loading — cascading: app defaults + user overrides

  mcp.py                   # MCP client lifecycle: McpSession, McpManager, LangChain tool wrapper

  _util.py                 # Shared constants (MAX_MESSAGE_SIZE)
  llm.py                   # LLM provider routing (ChatAnthropic/OpenAI/Google via model prefix)
  logging.py               # structlog configuration
  oauth.py                 # OAuth client for MCP HTTP servers (PKCE flows)
  scaffold.py              # `switchplane init` project scaffolding

examples/hello/        # Simple LangGraph graph (get_user → say_hello)
examples/weather/      # Long-running polling task (Open-Meteo weather watch, @command for coordinates)
examples/devops/       # Ops review: deterministic pandas analysis + LLM summary
examples/chatbot/      # Interactive LLM chat with interrupts
tests/                     # Test directory (pytest)
```

## Key design rules

- **The control plane owns task/event persistence.** Agents write only checkpoint data via a separate WAL-mode connection.
- **The control plane never runs domain logic.** All user code runs inside agent subprocesses.
- **Tasks are first-class.** Agents are just execution hosts. Every task has an ID, status, events, and stored results.
- **LangGraph-native.** Do not introduce generic workflow abstractions. Tasks use LangGraph StateGraph directly.
- **One app per runtime.** Each Application gets its own daemon, database, and socket.

## Application entry point

`Application(name="myapp")` creates an app. `app.discover_agents("myapp.agents")` registers discovery roots. `app.run()` discovers agents, builds the CLI, and starts it. The `name` determines the runtime directory (`~/.myapp/`). The CLI entry point is registered via `[project.scripts]` in pyproject.toml.

## Configuration

Two-layer cascading TOML config. App-bundled defaults (specified via `Application(default_config=Path(...))`) are deep-merged with user-level overrides at `~/.{app_name}/config.toml`. User config wins on conflict. Apps ship base URLs, model defaults, and agent settings; users provide personal config like API keys. Pydantic model: `AppConfig` with `LLMConfig` and per-agent dicts. Config is passed to agent subprocesses via the `execute_task` command payload and available as `ctx.config` in the agent runtime.

## How to run

```bash
# Setup
uv venv .venv && source .venv/bin/activate
uv pip install -e . -e examples/hello -e examples/weather -e examples/devops -e examples/chatbot

# Run a task (streams events inline, Ctrl+C detaches)
hello run example hello --name Alice

# Run detached
weather run weather watch -d

# Bare app invocation opens the full-screen TUI (auto-discovers running tasks)
hello

# Operator commands
hello runtime start|stop|status
hello task list|show|cancel|follow|resume|clear
hello agent list

# Send command to running task
weather task <task_id> <command> [--key value ...]
```

## CLI and TUI

**CLI** is always plain-text. `run`, `follow`, and `resume` stream events inline and support interactive command input (stdin reader thread). The TUI is **only** launched on a bare `<app>` call with a TTY. Non-TTY invocations always get plain text.

**TUI** (`tui.py`) is a full-screen prompt_toolkit app with tab-based navigation. Tab `[0] system` is always present and receives daemon command output. Task tabs start at `[1]`. The shared arg parser `_parse_kv_args` lives in `tui.py` and is imported by `cli.py`.

**Input prefixes** use a three-tier scheme designed to reserve freeform text for future interactive LLM tasks:
- `:` prefix → daemon commands: `:run`, `:task follow <id>`, `:runtime status`, `:agent list`, `:help`
- `/` prefix → task commands on the focused task: `/set_location --lat 40.7`
- Plain text → reserved for future freeform LLM interaction (currently shows a hint)

In CLI attached mode (`run`/`follow`/`resume`), only `/` task commands are supported inline. Daemon commands are not available — detach with Ctrl+C and use CLI subcommands directly.

**Event buffers** are capped at a configurable `[tui] max_buffer_lines` (default 2,000) per tab to bound both memory and per-frame render cost in long-running sessions. Spinner tick interval and refresh-debounce window are also configurable; see `TuiConfig` in `switchplane/config.py`.

## Build system

- **uv** for package management
- **hatchling** build backend
- No global CLI entry point — each app defines its own via `[project.scripts]`
- Virtual environment at `.venv/`

## Dependencies

click, pydantic v2, aiosqlite, langgraph, prompt_toolkit, structlog. Optional: `langchain-core` via `switchplane[llm]`; `mcp` + `langchain-core` via `switchplane[mcp]`. Example-specific: langchain-anthropic (devops, chatbot), httpx (weather), pandas + numpy (devops).

## IPC protocols

**CLI ↔ Control Plane:** Unix socket at `~/.{app_name}/runtime.sock`. Messages are 4-byte big-endian length prefix + JSON body. Types: `CliRequest` / `CliResponse`.

**Agent ↔ Control Plane:** Bidirectional over a per-agent Unix socketpair (`socket.socketpair(AF_UNIX)`). The CP creates the pair, passes one fd to the child via `--ipc-fd` + `pass_fds`. Same 4-byte length-prefixed JSON framing as CLI protocol. CP sends `AgentCommand`, agent sends `AgentEvent`. Agents can also send `AgentRequest` to the CP and receive `AgentResponse` — used for cross-task operations (submit, query, notify). This allows mid-execution cancel/shutdown — the agent runs a command listener concurrently with task execution. stdout/stderr are freed for normal logging.

## Database

SQLite at `~/.{app_name}/state.db` with WAL mode. Tables: `agents`, `tasks`, `events`, `checkpoints`, `checkpoint_writes`. Use Pydantic v2 `model_dump_json()` / `model_validate_json()` for JSON columns.

## Adding a task

1. Create a task module at `<app>/agents/<agent>/tasks/<task>.py`
2. Subclass `Task` from `switchplane` — set `name`, `description`, and optionally `mode` (`"ephemeral"` or `"long_running"`) class attributes
3. Declare parameters as class attributes using `Field()` from `switchplane` (re-exported from Pydantic). Parameters are validated before execution and set as instance attributes.
4. Implement `async def run(self, ctx: AgentContext)` — access params via `self.<param>`, build a LangGraph `StateGraph`, compile, `ainvoke`, then `ctx.complete(result)`
5. Optionally add `@command`-decorated methods for runtime commands on long-running tasks. Command parameters are typed and auto-coerced.
6. Optionally declare `mcp_servers: ClassVar[list[str]] = ["server-name"]` on the task class to specify which MCP servers the task needs. Only declared servers are started for that task. If not set, all agent-level servers are used.
7. Discovery auto-registers the task from the `tasks/` subpackage — no need to declare it in `AgentSpec`

## MCP integration

MCP servers are registered at the app level via `McpServerConfig` (provide `command` for stdio, `url` for HTTP — transport is inferred). Agents declare which servers they need via `AgentSpec.mcp_servers`. The agent runtime manages client lifecycle (spawning stdio processes or connecting to HTTP endpoints) and exposes tools via `ctx.mcp` (raw sessions) and `ctx.mcp_tools()` (LangChain `StructuredTool` wrappers). MCP is an optional dependency: `pip install switchplane[mcp]`. Tasks can also declare `mcp_servers` at the class level to restrict which servers are started for that specific task (instead of inheriting all servers from the agent).

## Cross-task coordination

Tasks can spawn child tasks, wait for their completion, and send notifications to sibling tasks via `AgentContext`:

- `ctx.submit_task(agent_name, task_name, params)` → submits a child task (linked via `parent_task_id`), returns the new `task_id`
- `ctx.get_task(task_id)` → returns current task record and events
- `ctx.wait_for_task(task_id)` → polls until terminal state, returns task record
- `ctx.wait_for_tasks(task_ids)` → parallel wait on multiple tasks
- `ctx.notify_task(task_id, payload)` → send a notification to another running task
- `ctx.wait_for_notification(timeout)` → block until a notification arrives

These requests travel over the existing agent↔CP socketpair as `AgentRequest`/`AgentResponse` messages. The control plane routes submissions and notifications across agents.

## Checkpoint and resume

Tasks opt into checkpointing by passing `ctx.checkpointer` to `graph.compile(checkpointer=ctx.checkpointer)` and using `ctx.task_id` as the LangGraph thread ID. The checkpointer is a `SqliteCheckpointSaver` backed by the app's `state.db` — each agent subprocess opens its own connection in WAL mode. LangGraph saves state after each node; on resume, it finds the existing checkpoint by thread ID and continues from the last completed node.

Resume flow: CLI sends `resume_task` request → control plane validates terminal status (failed/cancelled/completed) → resets to PENDING → re-launches with the same task ID. The `task resume <task_id>` CLI command handles this. Tasks that don't use `ctx.checkpointer` run without checkpointing and will re-execute from the beginning on resume.

## Testing

```bash
make test
```

## Style

- Python 3.12+ type annotations (use `X | None` not `Optional[X]`)
- Pydantic v2 (`model_dump()` not `.dict()`, `model_validate()` not `parse_obj()`)
- asyncio throughout the control plane; sync only in CLI client and daemon bootstrap
- Minimal abstractions — prefer direct code over premature generalization

---
> Source: [salesforce-misc/switchplane](https://github.com/salesforce-misc/switchplane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
