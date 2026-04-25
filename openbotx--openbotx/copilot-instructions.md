## openbotx

> OpenBotX is an AI agent orchestration platform. Python backend (FastAPI), Vue 3 frontend (PrimeVue), real-time WebSocket communication.

# AGENTS.md

OpenBotX is an AI agent orchestration platform. Python backend (FastAPI), Vue 3 frontend (PrimeVue), real-time WebSocket communication.

## Build and Run

```bash
# Setup (first time)
make setup
source .venv/bin/activate

# Run backend (dev mode with reload)
make dev

# Run frontend (dev mode, separate terminal)
make web-client-dev

# Build frontend for production (output goes to openbotx/web_client/)
make web-client-build

# Lint and format
make lint
make format
```

## Code Standards

### General Rules

- Follow the existing patterns, architecture, layout, and visual style already in the project. Consistency is paramount.
- Write professional, clean code using industry best practices. No workarounds, no fallbacks, no hacky solutions, no legacy code.
- No unexpected behavior from catch-all `else` branches for unknown cases. Handle only known cases explicitly.
- Do not create documentation files (`.md`), migrations, scripts, or tests unless explicitly asked.
- Do not run commands — edit files directly.
- Code and comments must be in English.

### Python

- Ruff linter, line length 100, target Python 3.11. Rules: E, F, I, N, W, UP (ignore E501).
- `__init__.py` files must be completely empty — no code, no imports, nothing.
- Do not use `TYPE_CHECKING` / `if TYPE_CHECKING` import pattern.
- Only add comments where truly essential (to save tokens). Single-line `#` comments must be lowercase. Block/docstring comments use normal casing.
- Validate imports with: `python -c "from openbotx.server.app import create_app"`.

### Frontend

- Vue 3 Composition API (`<script setup>`), Pinia stores as composable functions (`defineStore` with `setup()` syntax).
- Follow the existing PrimeVue component usage, CSS variable patterns, and layout conventions.

## Architecture Overview

The system follows a **message bus pattern** where all components communicate through async queues, not directly.

```
Channels (Web/Telegram) → MessageBus (inbound) → Orchestrator → AgentLoop → LLM Provider
                                                                    ↓
                                                              Tool execution
                                                                    ↓
                        MessageBus (outbound) ← AgentLoop ← Tool results / final response
                              ↓
                        ChannelManager → Channels → User
```

### Key Relationships

**Message flow:** Every user message becomes an `InboundMessage` → goes through `MessageBus.inbound` → `Orchestrator` picks it up → routes to the right `AgentLoop` (via `AgentClassifier` if multi-agent) → `AgentLoop` calls LLM + tools in a loop → publishes `OutboundMessage` to `MessageBus.outbound` → `ChannelManager` routes it back to the originating channel.

**One AgentLoop per agent:** Each agent defined in config gets its own `AgentLoop` instance with its own `LLMProvider`, `ToolRegistry`, `PathResolver`, workspace directory, and model. They share the same `MessageBus`, `SessionManager`, and `TaskManager`.

**Sessions tie to channels, not agents:** Session key = `{channel}:{chat_id}`. When multiple agents handle the same chat, they share the session history. The `AgentClassifier` reads this history (with `[Agent: name]` prefixes) to maintain conversation continuity.

**Tasks track everything:** Every inbound message creates a `Task` (TODO → DOING → DONE/ERROR). Tasks provide the real-time observability layer. The frontend task board reads tasks via REST and receives updates via WebSocket events.

**Tools are the agent's hands:** The LLM decides which tools to call. `ToolRegistry` validates parameters, executes, and returns string results. Each tool manages its own output size limits. Failed tools get a recovery hint appended.

**Subagents are fire-and-forget:** `SpawnTool` creates an `asyncio.Task` that runs independently. When done, the subagent publishes its result back to the inbound queue as a new message. The main agent picks it up in a future turn and connects it to the original conversation via session history.

**EventDispatcher decouples broadcasting:** The agent loop broadcasts events (`chat:thinking`, `chat:tool_use`, `task:created`, etc.) through `EventDispatcher`, which routes to `WebSocketManager`. Components never call WebSocket directly.

**live_state is transient runtime data:** Both `Session` and `Task` have a `live_state` dict populated during execution (tool_uses, agent_name). It's returned via API but never saved to JSONL. Cleared when processing completes.

## Package Structure

```
openbotx/
├── agent/                  # AI agent orchestration
│   ├── orchestrator.py     # Consumes inbound bus, routes to agents
│   ├── classifier.py       # LLM-based agent selection (multi-agent only)
│   ├── loop.py             # Main agentic loop: LLM → tools → repeat
│   ├── context.py          # System prompt assembly (SOUL.md, USER.md, memory, skills)
│   ├── memory.py           # MEMORY.md / HISTORY.md read/write, consolidation prompts
│   ├── skills.py           # SKILL.md discovery from builtin + workspace dirs
│   └── subagent.py         # Background agent spawning with restricted tools
│
├── bus/                    # Async message bus
│   ├── queue.py            # MessageBus: inbound + outbound asyncio.Queue
│   ├── events.py           # InboundMessage, OutboundMessage dataclasses
│   └── dispatcher.py       # EventDispatcher: broadcast events to handlers
│
├── server/                 # FastAPI application
│   ├── app.py              # ServerFactory (dependency creation), lifespan, create_app
│   ├── websocket.py        # WebSocketManager + endpoint (auth via query param)
│   ├── auth.py             # JWT middleware (protects /api/*)
│   └── routes/             # REST API
│       ├── chat.py         # Send messages, list/manage sessions
│       ├── tasks.py        # Task CRUD, state management
│       ├── files.py        # File tree, read, write, delete, upload, download
│       ├── skills.py       # List, load, update skills
│       ├── tools.py        # List tool definitions
│       ├── agents.py       # Agent listing
│       ├── channels.py     # Channel start/stop/status
│       ├── providers.py    # Provider listing
│       ├── scheduler.py    # Cron job CRUD
│       ├── config.py       # Read/update config, YAML editor, restart
│       ├── system.py       # System info (OS, CPU, RAM, disk, GPU)
│       └── auth.py         # Login endpoint
│
├── tools/                  # Agent tools (what the LLM can call)
│   ├── base.py             # Abstract Tool class
│   ├── registry.py         # ToolRegistry: register, lookup, execute, error hints
│   ├── filesystem.py       # read_file, write_file, edit_file, list_dir
│   ├── shell.py            # exec (with safety guards)
│   ├── web.py              # web_search (Brave), web_fetch (readability)
│   ├── http_client.py      # http_client (full HTTP + download/upload)
│   ├── rss.py              # rss_reader (RSS 2.0 + Atom)
│   ├── browser.py          # browser (Chrome CDP, multi-tab)
│   ├── message.py          # message (send to user, rate-limited per turn)
│   ├── spawn.py            # spawn (create subagent)
│   ├── cron.py             # cron (add/list/remove scheduled jobs)
│   ├── memory_tool.py      # memory_save, memory_read, memory_search
│   └── image.py            # generate_image (configurable provider)
│
├── channels/               # Communication channels
│   ├── base.py             # BaseChannel interface
│   ├── manager.py          # ChannelManager: outbound dispatch, routing
│   └── telegram.py         # TelegramChannel (polling, media, typing indicator)
│
├── providers/              # LLM provider abstraction
│   ├── base.py             # LLMProvider interface, LLMResponse dataclass
│   ├── litellm_provider.py # LiteLLM wrapper (prompt caching, sanitization)
│   └── registry.py         # ProviderSpec definitions (anthropic, openai, etc.)
│
├── config/                 # Configuration
│   ├── schema.py           # Pydantic models (Config, AgentConfig, etc.)
│   └── loader.py           # YAML loading with ${ENV_VAR} expansion
│
├── session/
│   └── manager.py          # SessionManager: JSONL persistence, in-memory cache
│
├── tasks/
│   ├── models.py           # Task dataclass, TaskState enum
│   └── manager.py          # TaskManager: create, update, broadcast events
│
├── storage/                # File storage backends
│   ├── base.py             # StorageProvider interface
│   ├── local.py            # Local filesystem
│   └── s3.py               # AWS S3
│
├── cron/
│   ├── service.py          # CronService: 5-second tick loop
│   └── types.py            # CronJob, CronSchedule (at/every/cron)
│
├── heartbeat/
│   └── service.py          # HeartbeatService: reads HEARTBEAT.md periodically
│
├── helpers/
│   ├── path.py             # PathResolver: workspace-scoped path resolution
│   ├── oauth.py            # OAuth 1.0a signature generation (HMAC-SHA1)
│   ├── text.py             # humanize(), describe_tool_use()
│   ├── config.py           # Configuration helper utilities
│   └── transcription.py    # Audio transcription (faster-whisper)
│
├── skills/                 # Built-in SKILL.md files (one folder per skill)
├── cli/commands.py         # CLI: init, start, version
└── version.py              # __version__
```

## Frontend Structure

```
web_client/src/
├── pages/                  # Route-level views
│   ├── ChatPage.vue        # Chat interface with session sidebar
│   ├── TaskBoard.vue       # Kanban board (TODO/DOING/DONE/ERROR columns)
│   ├── FilesPage.vue       # File manager with type-aware editors
│   ├── SkillsPage.vue      # Skill cards with editor dialog
│   ├── ToolsPage.vue       # Tool cards with parameter schema
│   ├── SchedulerPage.vue   # Cron job management
│   ├── SettingsPage.vue    # Tabbed config (Info, Bot, Channels, Storage, etc.)
│   └── LoginPage.vue       # Authentication
│
├── components/
│   ├── chat/               # ChatMessages, ChatInput, SessionList, ToolUseIndicator
│   ├── files/              # FileTree, MarkdownEditor, TextEditor, MediaPreview, FileDownload
│   ├── tasks/              # TaskCard, TaskColumn
│   └── common/             # AppSidebar
│
├── stores/                 # Pinia state (composable setup syntax)
│   ├── chat.js             # Messages, sessions, streaming, tool use tracking
│   ├── tasks.js            # Task map, activeTools map, agent filtering
│   ├── websocket.js        # WebSocket connection, event routing to stores
│   ├── agents.js           # Agent list
│   ├── auth.js             # JWT token, login/logout
│   ├── channels.js         # Channel status
│   └── config.js           # Platform configuration
│
├── composables/useApi.js   # Fetch wrapper with JWT auth headers
├── layouts/DefaultLayout.vue
├── router/index.js
└── main.js
```

**Frontend-backend communication:** REST API for CRUD, WebSocket for real-time events. The `websocket.js` store connects on login and routes events to the appropriate stores (chat events → chat store, task events → tasks store, etc.).

## Data Flow Patterns

**Inbound message lifecycle:**
`Channel → InboundMessage → MessageBus.inbound → Orchestrator → AgentClassifier (if multi-agent) → AgentLoop.process_message → Task(DOING) → Session.get_history → ContextBuilder.build_system_prompt → _run_agent_loop → LLM ↔ Tools (loop) → Session.add_message → OutboundMessage → MessageBus.outbound → ChannelManager → Channel → User → Task(DONE)`

**Tool execution in the loop:**
`LLM response has tool_calls → for each: ToolRegistry.execute(name, args) → broadcast chat:tool_use → append to session.live_state + task.live_state → add tool result to messages → next LLM call`

**Session persistence:**
Sessions are JSONL files in `workspace/sessions/`. The `SessionManager` keeps an in-memory cache — same Python object is shared between the agent loop and API handlers. `live_state` lives only in memory (never written to JSONL), making it immediately visible to API requests during execution.

**Task persistence:**
Tasks are JSONL in `workspace/tasks.jsonl`. On server restart, DOING tasks are recovered (main → reset to TODO and re-queued, subagent → set to ERROR).

## Configuration

Config lives in `config.yml` at the project root. Loaded by `openbotx/config/loader.py` with `${ENV_VAR}` expansion. Schema defined in `openbotx/config/schema.py` (Pydantic models).

Key sections: `bot`, `server`, `agents` (list of named agents), `auth`, `providers`, `channels`, `tools`, `storage`, `image`, `heartbeat`, `cron`, `classifier`.

Each agent in `agents` has: `model`, `workspace`, `description`, `instructions`, `tools` (whitelist), `params` (max_iterations, max_tokens, temperature, memory_window).

## Security Model

- `PathResolver` restricts file access to workspace + public dirs (per-agent).
- `ExecTool` blocks destructive commands (rm -rf, format, dd, shutdown, fork bombs) and path traversal.
- Subagents cannot: send messages, spawn more subagents, create cron jobs, or modify memory.
- JWT auth on all `/api/*` routes. WebSocket auth via query param.

## Conventions

- Bootstrap files (`SOUL.md`, `USER.md`, `AGENTS.md`, `TOOLS.md`) live at the project root and are injected into every system prompt. They are read on every message (changes take effect immediately).
- Skills are `SKILL.md` files with YAML frontmatter in `openbotx/skills/` (builtin) or `workspace/skills/` (project). Project skills override builtin ones with the same name.
- Memory files (`MEMORY.md`, `HISTORY.md`) live in `workspace/memory/`. `MEMORY.md` is included in the system prompt. `HISTORY.md` is append-only archival.
- All media goes to `public/media/YYYY/MM/DD/` via `media_path()` from `openbotx/helpers/path.py`.
- The built web client is packaged inside `openbotx/web_client/` in the Python distribution. Served as SPA at `/app/` with catch-all fallback.

---
> Source: [openbotx/openbotx](https://github.com/openbotx/openbotx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
