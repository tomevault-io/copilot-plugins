## claude-telegram-agent

> Personal AI agent with memory, tasks, cron, web UI, and Telegram integration


# Claude Agent OS

## Skills

### Memory Management
Persistent file-based memory with YAML frontmatter. Supports types: user, feedback, project, reference. Memories are indexed, searchable, and survive across sessions.

### Task Tracking
Kanban-style task management with statuses (todo, in-progress, done, archived), priorities, and filtering.

### Cron Scheduling
Define recurring agent jobs as YAML files. Jobs run on schedule via APScheduler, spawning Claude subprocesses.

### Web Dashboard
Password-protected local web UI (FastAPI + Jinja2) with chat, tasks, memory browser, cron manager, logs viewer, and settings editor.

### Telegram Integration
Two-way messaging via Telegram Bot API. User allowlist for access control. Supports text messages and file attachments.

### Sub-Agent Spawning
Pool-based concurrent Claude subprocess management for parallel workloads.

### Webhook Triggers
HTTP POST endpoints to trigger agent runs programmatically.

### macOS Daemon
LaunchDaemon or Homebrew service for always-on background operation that survives reboots.

---
> Source: [travis-burmaster/claude-telegram-agent](https://github.com/travis-burmaster/claude-telegram-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
