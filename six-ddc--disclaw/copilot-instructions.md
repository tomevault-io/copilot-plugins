## disclaw

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Disclaw is a Discord bot that bridges Discord with Claude Code via the Agent SDK, enabling thread-based AI conversations with multimodal support. Architecture: `Discord Gateway → Bot → In-process JobRunner → Claude Agent SDK`. Single-process, no external dependencies beyond Discord. Runs locally with no exposed ports (outbound Discord gateway only).

## Commands

```bash
# Install dependencies
bun install

# Service management (auto-detects Linux systemd / macOS launchd)
make install       # Register as service (auto-detects project dir and bun path)
make start         # Start the service
make stop          # Stop the service
make restart       # Restart the service
make status        # Show service status
make logs          # Follow service logs
make deploy        # Type-check + restart
make uninstall     # Remove service + unlink skills
make link-skills   # Symlink project skills to ~/.claude/skills/
make unlink-skills # Remove skill symlinks from ~/.claude/skills/

# Development
make dev           # Start with hot reload (bun --watch, not as service)
make typecheck     # Type checking (bunx tsc --noEmit)
```

There are no tests or linting configured.

## Architecture

**Request flow:** User @mentions bot in Discord → `bot.ts` creates a thread, stores thread→session mapping in SQLite (session ID empty), submits job to `runner.ts` → runner calls `claude-client.ts` which uses `@anthropic-ai/claude-agent-sdk` `query()` to get an async iterator of SDKMessages → SDK auto-generates session ID, saved to DB on init message → messages are converted via `message-converter.ts` and rendered to Discord via `discord-sender.ts`. Follow-up messages in the thread use `resume: sessionId` to continue the session.

**Session ID lifecycle:** The bot never generates session UUIDs — the SDK auto-generates them. On each query, the SDK sends an init message (`type: 'system', subtype: 'init'`) containing `session_id`. The runner compares this with the stored ID; if different (new session), it saves the SDK ID to DB and sends a "New session" / "Forked session" notification embed. Ephemeral sessions (`persistSession: false`, e.g. cron) skip this.

**Working directory resolution (fallback chain):** `[/path]` message prefix override → `mapping.working_dir` (thread-level) → channel config → `CLAUDE_WORKING_DIR` env → `~/.disclaw`. Centralized in `working-dir.ts`. Thread-level override is set via `/disclaw cd` in a thread, which also clears the session (new directory = fresh session).

**Key source files:**
- `src/bot.ts` — Discord event routing (MessageCreate, MessageReactionAdd/Remove, interaction dispatch). No command handler logic — delegates to `interactions.ts`
- `src/interactions.ts` — All `/disclaw` slash command handlers. Thread-only commands use `requireThreadSession()` guard. Config modal (model/permission/display) uses `LabelBuilder` + `ModalBuilder`
- `src/dir-picker.ts` — Interactive button-based directory browser for Discord. Navigation: subdirectory buttons (paginated), Up/Prev/Next, Select/Cancel. 2-minute timeout
- `src/runner.ts` — In-process job runner with concurrency control (semaphore, default 10), per-thread job serialization with message batching/merging, retry with exponential backoff. Tracks active `Query` objects per thread for interrupt support. Detects session changes from SDK init messages. Stall detection with auto-resume for persistent sessions
- `src/claude-client.ts` — SDK wrapper; calls `query()` and streams SDKMessages via `onMessage` callback. Supports `model`, `forkSession`, `resumeSessionAt`, `canUseTool`, `permissionMode`, `persistSession` options
- `src/user-input.ts` — Handles SDK `canUseTool` callback via Discord interactive components. Manages AskUserQuestion (buttons/selects/modals) and tool approval UI (Allow/Deny/Always Allow). Per-thread auto-approved tools. 5-minute timeout
- `src/attachment-handler.ts` — Multimodal content extraction: images (PNG/JPEG/GIF/WebP, max 20MB, max 5), PDFs (max 20MB), text files (max 100KB), and reply references. Converts Discord attachments to Claude API content blocks. Type definitions for `ContentBlock` and `MultimodalPrompt`
- `src/message-converter.ts` — Transforms raw SDKMessages into normalized `ClaudeMessage` objects (text, tool_use, tool_result, thinking, system, permission_denied, task_started, task_notification, task_progress, tool_progress, tool_summary, other)
- `src/discord-sender.ts` — Renders `ClaudeMessage` objects as Discord embeds/messages with rich formatting. Delegates tool embed construction to `tool-embeds.ts`. Handles completion stats, session change notifications, and auto-deleting status messages
- `src/history.ts` — Paginated session history viewer. Uses SDK `getSessionMessages()`, renders as Discord embed with ◀/▶ navigation buttons. Used by fork, resume, and rewind
- `src/discord.ts` — Discord REST API helpers (send, edit, typing indicators), markdown splitting with table flattening
- `src/db.ts` — SQLite database for thread→session mappings, channel configs, cron jobs, and pager messages. Thread mappings include: session_id, working_dir, model, fork_from, permission_mode, display_mode, title
- `src/cron.ts` — Scheduled task system: `CronScheduler` class managing job lifecycle (register, pause, resume, delete, runNow). Auto-pauses after 3 consecutive failures
- `src/cron-buttons.ts` — Cron control panel UI: persistent Discord buttons for Pause/Resume, Run Now, Verbose toggle, Delete
- `src/mcp-server.ts` — MCP server factory creating per-query SDK MCP servers. Exposes 16 tools: `cron_create`, `cron_list`, `cron_delete`, `cron_update`, `cron_run_now` (cron management), `discord_send`, `discord_edit`, `discord_get`, `discord_list`, `discord_create_thread`, `discord_set_title`, `discord_react`, `discord_unreact`, `discord_delete`, `discord_channels`, `discord_threads` (Discord API). File validation with extension whitelist and 25MB size limit
- `src/tool-embeds.ts` — Shared tool embed building logic used by both `discord-sender.ts` (verbose mode) and `tool-pager.ts` (pager mode). Provides `buildToolUseEmbed()`, `buildToolResultField()`, `truncateContent()`, `formatToolName()` with specialized formatting for Agent, Edit, Write, Read/Glob/Grep, TodoWrite, Bash, cron tools, and generic tools
- `src/tool-pager.ts` — Two-phase paginated display for tool calls/thinking/text in a single navigable Discord embed. Phase 1 (live): in-memory with debounced updates during job execution. Phase 2 (persistent): SDK-backed on-demand page rendering after completion. Supports reaction-triggered button restore/hide and auto-upgrade across bot restarts. Buttons auto-strip after 30s or when next pager appears
- `src/context-builder.ts` — XML-formatted prompt builder with Discord context injection. Formats user messages as structured XML with sender attribution, timestamps, reply references, and attachments. Prepends `<context>` with channel info, forum post content, thread history, and cron job metadata when adopting threads or starting cron sessions
- `src/paths.ts` — XDG Base Directory compliant default paths. Exports `DEFAULT_DB_PATH` (`$XDG_DATA_HOME/disclaw/threads.db`), `DEFAULT_WORKING_DIR` (`~/.disclaw`), `DEFAULT_LOG_DIR` (`$XDG_STATE_HOME/disclaw/logs`)
- `src/working-dir.ts` — Centralized working directory resolution. Implements the full fallback chain: `[/path]` message prefix → thread mapping → channel config → `CLAUDE_WORKING_DIR` env → `~/.disclaw`. Also provides `validateWorkingDir()` for security allowlist enforcement
- `src/types.ts` — Centralized type definitions for `PermissionMode` and `DisplayMode`. Single source of truth for mode values, labels, descriptions, and type guards used across the codebase
- `src/logger.ts` — Centralized logging with pino; outputs to both console and daily log files (`$XDG_STATE_HOME/disclaw/logs/YYYY-MM-DD.log`)

**Slash commands (`/disclaw <subcommand>`):**
- `cd` — Set working directory (channel default or thread override; interactive dir picker)
- `clear` — Clear conversation context, start fresh (thread-only)
- `interrupt` — Interrupt current Claude processing (thread-only)
- `config` — Configure model, permission mode, and display mode via modal with select menus (thread-only)
- `fork` — Fork conversation into a new thread (thread-only)
- `resume` — Resume a previous session via select menu (any location)
- `cron` — List all scheduled tasks (any location)

**Data stores:**
- SQLite (`$XDG_DATA_HOME/disclaw/threads.db`, default `~/.local/share/disclaw/threads.db`) — thread/session mappings (incl. working_dir, model, permission_mode, display_mode), channel configs, cron_jobs, pager_messages
- No message content is stored (privacy-first design)

## Runtime & Build

- **Runtime:** Bun (TypeScript executed natively, no build/compile step)
- **Module system:** ESM (`"type": "module"`)
- **TypeScript:** Strict mode with `noUncheckedIndexedAccess` and `noImplicitOverride` enabled; `noEmit: true` (type checking only)
- **Key dependencies:** discord.js, @anthropic-ai/claude-agent-sdk, croner (cron scheduling), zod (schema validation), pino / pino-pretty (structured logging)

## Service

Cross-platform service management in `service/`. Auto-detects Linux (systemd) / macOS (launchd):
- `manage.sh` — Cross-platform service manager (install/uninstall/start/stop/restart/status/logs)
- `disclaw.service.template` — Linux systemd unit template (`__PROJECT_DIR__`, `__BUN_PATH__` placeholders)
- `com.disclaw.bot.plist` — macOS LaunchAgent plist template (`__BUN_PATH__`, `__WORKING_DIR__`, `__LOG_DIR__` placeholders)

`make install` generates the platform-specific service file from the template. Linux: writes to `/etc/systemd/system/` (requires sudo). macOS: writes to `~/Library/LaunchAgents/` (user-level, no sudo).

## Logging

Uses `pino` (`src/logger.ts`). Output goes to both console and `$XDG_STATE_HOME/disclaw/logs/YYYY-MM-DD.log` (default `~/.local/state/disclaw/logs/`). Override with `LOG_DIR` env var. When running as a service, logs are also available via `make logs` (Linux: journalctl, macOS: tail logs/). discord.js has built-in rate limit retry — do not add custom retry wrappers on top.

## Environment Variables

Required: `DISCORD_BOT_TOKEN`. Optional: `CLAUDE_WORKING_DIR` (default: `~/.disclaw`), `DISCLAW_ALLOWED_DIRS` (comma-separated security allowlist), `LOG_DIR` (default: `$XDG_STATE_HOME/disclaw/logs`), `DISCLAW_PERMISSION_MODE` (default: `default`; options: `default`, `dontAsk`, `acceptEdits`, `bypassPermissions`, `plan`), `SHOW_LINK_PREVIEWS` (show URL embeds in bot messages), `TZ`. See `.env.example`.

---
> Source: [six-ddc/disclaw](https://github.com/six-ddc/disclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
