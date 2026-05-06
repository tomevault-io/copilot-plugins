## neoclaw

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
bun install          # Install dependencies
bun start            # Start daemon (self-daemonizes to background, logs to ~/.neoclaw/logs/neoclaw.log)
bun onboard          # Generate ~/.neoclaw/config.json config template
bun run dev          # Run with --watch (auto-restart on file changes)
bun run typecheck    # Type-check without emitting (bun run --bun tsc --noEmit)
bun run lint         # Lint with ESLint + Prettier
```

### CLI (citty)

Entry point: `src/index.ts` → `src/cli/index.ts`. Uses [citty](https://github.com/unjs/citty) for CLI framework. Subcommands are lazily loaded from `src/cli/commands/`.

```bash
neoclaw start        # Start daemon (alias: bun start)
neoclaw stop         # Stop running daemon
neoclaw onboard      # Generate config template
neoclaw cron <sub>   # Cron job management (create, list, delete, update)
```

## Architecture

Gateway pattern. `Dispatcher` (`dispatcher.ts`) routes messages from **Gateways** (I/O adapters) to **Agents** (AI backends).

```
Gateway.start(dispatcher.handle)
  → InboundMessage + ReplyFn + StreamHandler → Dispatcher → Agent.stream() → AgentStreamEvent*
                                                                            ↓
                                                             streamHandler(stream) → Gateway renders
```

**Message flow:**
1. Gateway receives a raw platform event, parses it into `InboundMessage`, creates a `reply` closure and a `streamHandler` closure with protocol context bound
2. Gateway calls `dispatcher.handle(msg, reply, streamHandler)`
3. Dispatcher acquires a per-conversation `SerialQueue` lock to prevent race conditions
4. Checks for slash commands (`/clear`, `/new`, `/restart`, `/status`, `/help`) — always use non-streaming `reply()`
5. If `streamHandler` is provided and agent has `stream()`, uses streaming path; otherwise falls back to `agent.run()` + `reply()`
6. Gateway's `streamHandler` renders content progressively as events arrive

**Key interfaces** (in `*/types.ts`):
- `Agent`: `run()`, `stream?()`, `healthCheck()`, `clearConversation()`, `dispose()` — AI backend
- `Gateway`: `start(handler)`, `stop()`, `send()` — messaging platform adapter
- `ReplyFn`: `(response: RunResponse) => Promise<void>` — for slash commands and non-streaming fallback
- `StreamHandler`: `(stream: AsyncIterable<AgentStreamEvent>) => Promise<void>` — for progressive rendering
- `MessageHandler`: `(msg, reply, streamHandler?) => Promise<void>` — called by Gateway for each message
- `AgentStreamEvent`: `{ type: 'thinking_delta', text }` | `{ type: 'text_delta', text }` | `{ type: 'tool_use', name, input }` | `{ type: 'ask_questions', questions, conversationId }` | `{ type: 'done', response: RunResponse }`
- `InboundMessage`: `id`, `text`, `chatId`, `threadRootId?`, `authorId`, `authorName?`, `gatewayKind`, `chatType?` (`'private'` | `'group'`), `attachments?: Attachment[]`, `meta?`
- `RunResponse`: `text`, `thinking?`, `sessionId?`, `costUsd?`, `inputTokens?`, `outputTokens?`, `elapsedMs?`, `model?`

**Agents**: `ClaudeCodeAgent` uses Claude Code CLI via long-running subprocess with bidirectional JSONL streaming. Maintains one subprocess per `conversationId` (pooled, reaped after 10 min idle). After idle reap, session IDs are persisted in `~/.neoclaw/cache/sessions.json` so the next request **resumes** the same Claude session (`--resume <sessionId>`). Each conversation runs in its own workspace directory `~/.neoclaw/workspaces/<conversationId>` (`:` replaced with `_`); the directory is created on first use. Default model: `claude-sonnet-4-6`. Passes `NEOCLAW_CHAT_ID` and `NEOCLAW_GATEWAY_KIND` env vars to the subprocess for cron/context routing. Disallows `CronCreate`, `CronDelete`, `CronList` tools (handled via CLI instead).

The `stream()` method on `ClaudeCodeAgent` yields `AgentStreamEvent`s: `thinking_delta` and `text_delta` are emitted for each JSONL `content_block_delta`; `tool_use` events pass through tool call info; `ask_questions` events surface interactive permission/question prompts; a final `done` event carries the full `RunResponse` (including stats).

**File access security** (`agents/file-blocked-agent.ts`, `utils/file-guard.ts`): `createFileBlockedAgent()` wraps any Agent to enforce file access policies:
- **File blacklist**: Blocks reading/writing files matching configurable glob patterns (supports `*`, `**`, `?`, tilde expansion). Default blacklist covers `~/.claude/**`, `~/.config/claude/**`, `/etc/shadow`, `/etc/passwd`, `**/.env`, `**/credentials.json`, `**/secrets/**`, `~/.neoclaw/config.json`, `~/.neoclaw/config.json.backup`.
- **Group chat write restrictions**: For `chatType === 'group'`, blocks write operations (`rm`, `mv`, `cp`, `touch`, `mkdir`, `chmod`, `chown`, `ln`, `tee`, `>`, `>>`, Write/Edit/NotebookEdit tools) targeting paths outside the conversation's workspace directory.
- Intercepts `tool_use` stream events; in non-streaming mode returns error message instead of executing.

**Gateways**:

*FeishuGateway* (Feishu/Lark WebSocket). Handles:
- Reaction emoji lifecycle (`OneSecond` emoji added on receive, removed after reply)
- Reply threading (replies are sent as thread replies to the original message)
- Error reporting back to the user — all transparent to the Dispatcher
- **Streaming responses** via Feishu Card JSON 2.0 (cardkit API): card created lazily on first delta, updated progressively via `cardkit.v1.cardElement.content`, closed with `cardkit.v1.card.settings`. `print_strategy: 'delay'` gives typewriter effect on the client side
- **Thinking panel**: inserted dynamically via `insertThinkingPanel()` only when a `thinking_delta` arrives (not pre-created); collapsed automatically on `done`
- Non-streaming responses (slash commands, proactive sends) use Feishu Card JSON 1.0-style cards with optional collapsible Thinking panel, markdown body, stats note footer
- Media attachment download: images, files, audio, video, stickers are downloaded and passed as `Attachment[]` in `InboundMessage.attachments`; a `<media:type>` placeholder is appended to the message text
- Quoted/parent message fetching: when a message quotes another (`parent_id`), the quoted text is prepended as `[Replying to: "..."]`
- In group chats, the sender's display name is prepended to the message text for context
- `groupAutoReply`: chats listed in `feishu.groupAutoReply` receive replies without requiring an @mention

*WeworkWsGateway* (`gateway/wework/`): WeChat Work (企业微信) WebSocket gateway. Connects to `wss://openws.work.weixin.qq.com`. Features:
- Message deduplication via `seenMsgIds` set (max 10000 entries)
- Message debouncing (2000ms default) to batch rapid inputs
- Streaming responses with progressive updates
- Active stream tracking for proactive sends
- Statistics formatting: model, elapsed time, token counts, cost
- `groupAutoReply`: chats listed in `wework.groupAutoReply` receive replies without requiring an @mention

*DashboardGateway* (`gateway/dashboard/`): Web-based chat interface via WebSocket. Features:
- HTTP server with CORS support; WebSocket connections on `/ws` path
- Session management: create, list, switch sessions per client
- Message types: `register`, `message` (client → server); `response`, `stream_start`, `stream_delta`, `stream_end`, `error`, `sessions_update` (server → client)
- Auto-starts frontend dev server (npm/bun) when enabled
- Health check endpoint at `/health`

**Feishu sender** (`gateway/feishu/sender.ts`): Two card formats:
1. **Non-streaming** (JSON 1.0-style): `buildCard()` → `sendCard()` / `sendMarkdown()`
2. **Streaming** (JSON 2.0, cardkit): `buildStreamingCard()` → `createCardEntity()` → `sendCardByRef()` → `updateCardText()` / `insertThinkingPanel()` / `patchCardElement()` / `appendCardElements()` → `closeCardStreaming()`
   - Element IDs tracked in `STREAM_EL` constant (`thinking_panel`, `thinking_md`, `thinking_hr`, `main_md`, `stats_hr`, `stats_note`)
   - Stats footer uses `markdown` tag (JSON 2.0 does not support `note` tag)

**Session isolation**: Thread messages use key `${chatId}_thread_${threadId}` so threads don't share context with the main chat.

**Cron scheduling** (`src/cron/`): Manages scheduled jobs (one-time and recurring).
- `CronScheduler` (`scheduler.ts`): Polls every 30 seconds for due jobs. Matches 5-field cron expressions (min hour day month weekday). Prevents double-fires within 50 seconds. Injects conversation context into synthetic `InboundMessage`.
- `CronJob`: `{ id, label?, message, chatId, gatewayKind, conversationId, runAt? (ISO 8601 one-time), cronExpr? (5-field cron recurring), enabled, createdAt, lastRunAt? }`
- CLI (`src/cli/commands/cron.ts`, command: `neoclaw cron`): `create`, `list`, `delete`, `update` subcommands. Reads `NEOCLAW_CHAT_ID` and `NEOCLAW_GATEWAY_KIND` from environment. Cron CLI instructions are injected into the agent's system prompt.

**Daemon** (`daemon.ts`): Self-daemonizes on first launch (forks to background with `NEOCLAW_DAEMON=1` env var, redirects I/O to `~/.neoclaw/logs/neoclaw.log`). Uses PID file (`~/.neoclaw/cache/neoclaw.pid`) with SIGTERM takeover of existing instance (up to 10s grace, then SIGKILL). `/restart` command saves `~/.neoclaw/cache/restart-notify.json` (contains `{ chatId, gatewayKind }`), forks a new process, then aborts the current one; new process waits 5s for gateways to initialize, then delivers restart confirmation via `dispatcher.sendTo()` (retries up to 3 times with 3s delay). On startup, wraps agent with `createFileBlockedAgent()`, injects cron CLI instructions + file access restrictions into system prompt, validates at least one gateway is configured, and starts `CronScheduler`.

**Config** (`config.ts`): Loaded from env vars > `~/.neoclaw/config.json` > defaults. Env var prefix: `NEOCLAW_*` for agent/runtime, `FEISHU_*` for Feishu credentials, `WEWORK_*` for WeChat Work credentials.

Key config fields (`~/.neoclaw/config.json`):
```jsonc
{
  "agent": {
    "type": "claude_code",       // only supported value
    "model": "claude-sonnet-4-6", // Claude model override
    "summaryModel": "haiku",     // model for session summarization
    "systemPrompt": "...",        // extra system prompt
    "allowedTools": [],           // empty = --dangerously-skip-permissions
    "timeoutSecs": 600            // agent response timeout
  },
  "feishu": {
    "appId": "",
    "appSecret": "",
    "verificationToken": "",
    "encryptKey": "",
    "domain": "feishu",           // "feishu", "lark", or custom base URL
    "groupAutoReply": []          // chat IDs for auto-reply without @mention
  },
  "wework": {
    "botId": "",                  // 企业微信智能机器人 ID
    "secret": "",                 // Secret key
    "websocketUrl": "",           // optional custom WebSocket URL
    "groupAutoReply": []          // chat IDs for auto-reply without @mention
  },
  "dashboard": {
    "enabled": false,             // enable web dashboard
    "port": 3000,                 // HTTP/WebSocket port
    "cors": true                  // enable CORS
  },
  "mcpServers": {                 // MCP servers exposed to agents (hot-reloaded)
    "server-name": {
      "type": "stdio",            // "stdio" | "http" | "sse"
      "command": "npx",
      "args": ["-y", "@example/mcp-server"],
      "env": {}
    }
  },
  "fileBlacklist": [],            // glob patterns to block file access
  "skillsDir": "~/.neoclaw/skills", // skill directories (each with SKILL.md)
  "logLevel": "info",             // "debug" | "info" | "warn" | "error"
  "workspacesDir": "~/.neoclaw/workspaces"  // base dir; per-conversation subdirs created on demand
}
```

Env var overrides: `NEOCLAW_AGENT_TYPE`, `NEOCLAW_MODEL`, `NEOCLAW_SUMMARY_MODEL`, `NEOCLAW_SYSTEM_PROMPT`, `NEOCLAW_ALLOWED_TOOLS`, `NEOCLAW_TIMEOUT_SECS`, `NEOCLAW_LOG_LEVEL`, `NEOCLAW_WORKSPACES_DIR`, `NEOCLAW_SKILLS_DIR`, `NEOCLAW_FILE_BLACKLIST`, `NEOCLAW_CONFIG` (config file path), `NEOCLAW_DASHBOARD_ENABLED`, `NEOCLAW_DASHBOARD_PORT`, `NEOCLAW_DASHBOARD_CORS`, `FEISHU_APP_ID`, `FEISHU_APP_SECRET`, `FEISHU_VERIFICATION_TOKEN`, `FEISHU_ENCRYPT_KEY`, `FEISHU_DOMAIN`, `FEISHU_GROUP_AUTO_REPLY`, `WEWORK_BOT_ID`, `WEWORK_SECRET`, `WEWORK_WEBSOCKET_URL`, `WEWORK_GROUP_AUTO_REPLY`.

**MCP & Skills workspace sync** (`ClaudeCodeAgent._prepareWorkspace()`): Runs each time a new Claude Code subprocess starts. `_syncMcpServers()` hot-reloads `mcpServers` from the config file (not cached opts) and writes `<workspace>/.mcp.json`; the built-in `neoclaw-memory` MCP server is always injected alongside user-configured servers. `_syncSkills()` reads `skillsDir`, symlinks valid skill directories (containing `SKILL.md`) into `<workspace>/.claude/skills/`, and removes stale symlinks for deleted skills.

**Memory system** (`src/memory/`): Three-layer architecture exposed via a built-in stdio MCP server (`mcp-server.ts`):
- `MemoryStore` (`store.ts`): SQLite FTS5 index over markdown files. Content table + FTS5 virtual table with triggers for sync. Categories: `identity`, `knowledge`, `episode`. `reindex(memoryDir)` rebuilds from disk.
- `MemoryManager` (`manager.ts`): Tool handlers (`handleSearch`, `handleSave`, `handleList`, `handleRead`), session summarization (`summarizeSession`), periodic reindex (every 5 min via `startPeriodicReindex()`).
- `summarizer.ts`: Calls `claude --print` (haiku model, configurable via `agent.summaryModel`) to generate structured session summaries.
- **MCP Server** (`mcp-server.ts`): Standalone stdio MCP server that instantiates its own `MemoryStore` + `MemoryManager`. Automatically injected into each workspace's `.mcp.json` so Claude Code can directly call `memory_search`, `memory_save`, `memory_list`, `memory_read`. Receives `NEOCLAW_MEMORY_DIR` via environment variable.
- **Session summarization**: On `/clear` or `/new`, Dispatcher calls `summarizeSession()` which reads only new content from `.history/` (tracked via `.last-summarized-offset` marker), generates summary, saves to `episodes/`, updates index.
- **Storage layout**: `~/.neoclaw/memory/` — `identity/SOUL.md` (identity), `knowledge/*.md` (semantic: `owner-profile`, `preferences`, `people`, `projects`, `notes`), `episodes/*.md` (episodic), `index.sqlite` (FTS5 index). All `.md` files use the same frontmatter format (`title`, `date`, `tags`).
- Four tools: `memory_search` (query + optional category filter), `memory_save` (content + topic for knowledge, or category="identity" for identity/SOUL.md), `memory_list` (optional category filter), `memory_read` (read specific memory file).

## Conventions

- Full async/await — no sync blocking in async paths
- TypeScript strict mode with `noUncheckedIndexedAccess`
- Interfaces over class inheritance for loose coupling
- All runtime files (`logs/`, `cache/`, `workspaces/`, `skills/`, `memory/`) live under `~/.neoclaw/`; PID file at `~/.neoclaw/cache/neoclaw.pid`
- `Bun.spawn()` for subprocesses; `Bun.sleepSync()` only in daemon takeover loop

---
> Source: [amszuidas/neoclaw](https://github.com/amszuidas/neoclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
