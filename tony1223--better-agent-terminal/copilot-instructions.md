## better-agent-terminal

> - **NEVER** break existing features when implementing new ones.

# CLAUDE.md - Project Guidelines

## No Regressions Policy

- **NEVER** break existing features when implementing new ones.
- Before committing, verify ALL existing features still work — not just the new changes.
- Run the build (`npx vite build`) to confirm compilation succeeds.
- When modifying shared code (stores, IPC handlers, types), trace all consumers to ensure nothing breaks.

## Logging

- **Frontend (renderer)**: Use `window.electronAPI.debug.log(...)` instead of `console.log()`. This sends logs to the electron main process logger, which writes to disk.
- **Backend (electron)**: Use `logger.log(...)` / `logger.error(...)` from `./logger`.
- Do NOT use `console.log()` for debugging — use the logger so logs are persisted and visible in the log file.
- **Log file location**: `~/Library/Application Support/better-agent-terminal/debug.log`

## Sub-agent / Active Tasks Tracking

- The Claude Agent SDK does **NOT** reliably emit `task_started` / `task_progress` / `task_notification` system messages.
- We track Agent/Task tools from `tool_use` blocks directly in `session.activeTasks` (in `claude-agent-manager.ts`).
- `stopTask()` falls back to using `toolUseId` as `task_id` when no mapping exists.
- Tool results for Agent/Task must clean up `activeTasks` entries.

## React Rendering

- Use `flushSync` from `react-dom` for Agent/Task tool state changes (`setMessages` in `onToolUse` and `onToolResult`) to prevent rendering delays from React 18 batching during streaming.
- Do NOT use `flushSync` for regular tool calls — only for state changes that affect the active tasks bar visibility.

## Status Line

- Our status line implementation is superior to external alternatives (e.g., ccstatusline). Do not replace it.
- 13 configurable items with custom colors, zone alignment, and template-based config.
- Usage polling: Chrome session key (primary, lenient rate limits) → OAuth fallback (strict rate limits).

## Release

- **正式版**: `release new tag version` → 基於最新 tag 遞增 patch 版號，建立 tag 並 push
  - 例：目前 `v2.1.3` → 建立 `v2.1.4` tag
- **預覽版**: `release new pre tag version` → 基於最新 tag 遞增 patch 版號，加 `-pre.1` 後綴
  - 例：目前 `v2.1.3` → 建立 `v2.1.4-pre.1`
  - 若已有 `v2.1.4-pre.1` → 建立 `v2.1.4-pre.2`
- Tag 含 `-pre` 時 GitHub Release 自動標為 Pre-release，不更新 Homebrew tap
- Tag 不含 `-pre` 時為正式版，更新 Homebrew tap

---
> Source: [tony1223/better-agent-terminal](https://github.com/tony1223/better-agent-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
