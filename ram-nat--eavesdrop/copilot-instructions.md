## eavesdrop

> TUI for browsing openclaw session JSONL files. Built with Textual (Python).

# eavesdrop — Maintainer Guide

TUI for browsing openclaw session JSONL files. Built with Textual (Python).

---

## Purpose

Openclaw stores every agent session as a JSONL file under `~/.openclaw/agents/main-cloud/sessions/`. Each line is a typed event. This tool provides a two-panel browser: session list on the left, conversation thread on the right.

---

## JSONL Format

Each line is a JSON object. Relevant types:

| `type` | Notes |
|---|---|
| `session` | First line. Has `id`, `timestamp`, `cwd`. |
| `model_change` | Has `modelId`, `provider`. |
| `message` | Main content. `message.role` is `user`, `assistant`, or `toolResult`. |

**`message.role` values:**
- `user` — human turn; `content[].type` is `text`
- `assistant` — model turn; content can include `thinking`, `toolCall`, `text`; has `model`, `provider`, `usage`, `stopReason`
- `toolResult` — tool output; a top-level role, not nested inside a user message; has `toolCallId`, `toolName`, `isError`, `details`

**Notable fields:**
- `details.aggregated` on toolResult: openclaw-specific compact summary of large output — prefer over `content[].text` when present
- Assistant text wrapped in `<final>...</final>` is openclaw's response tag; rendered as a collapsible block

**File naming:** `<uuid>.jsonl` (active) and `<uuid>.jsonl.reset.<timestamp>` (closed) are both shown. `<uuid>.jsonl.deleted.<timestamp>` is excluded. `session_uuid(path)` extracts the UUID from any variant.

---

## Architecture

```
eavesdrop/
  parser.py          — JSONL reader; returns ParsedSession(meta, events[], error)
  cron_parser.py     — Cron data model (CronJob, CronRun, CronRunContext) and loaders
  app.py             — Textual App; two-panel layout, keybindings, CLI wiring
  __main__.py        — argparse entry point (--session, --dir, --cron)
  widgets/
    file_browser.py  — ListView of sessions with per-item metadata
    cron_browser.py  — Two-level ListView: jobs → runs for selected job
    conversation.py  — VerticalScroll; mounts turn widgets from parsed events
    turn.py          — Per-turn widgets for all content block types;
                       also CronRunHeader and DebugLogSection
```

**Data flow:** `scan_sessions()` → `session_summary()` (lightweight, for browser) or `parse_file()` (full parse, for conversation) → widget tree.

**Global toggle state** lives in `ConversationView` (`_show_thinking`, `_show_usage`, `_tools_expanded`) and is propagated to child widgets. Per-item toggle is handled by the widgets themselves via `Enter` when focused.

**Cron mode** is toggled with `C`. Both `FileBrowser` and `CronBrowser` are always mounted; only one is visible at a time. `CronBrowser` has two levels: jobs list (level 1) → runs list for the selected job (level 2). Selecting a run fires `CronBrowser.SessionRequested`, which the app handles by calling `conv.load_session(path, cron_context=...)`. A `CronRunContext` carries the job, run, and optional debug log path. When present, `_rebuild()` mounts `CronRunHeader` + `DebugLogSection` above the session content. `d` toggles the debug section.

**Cron data paths** (relative to openclaw root, default `~/.openclaw`):
- `cron/jobs.json` — job definitions
- `cron/runs/[jobId].jsonl` — run history per job
- `agents/main-cloud/sessions/` — session files (same dir as regular mode)
- `logs/openclaw-debug.log` — debug log entries

**Search state** also lives in `ConversationView` (`_search_matches`, `_search_index`, `_search_active`). `_block_text()` in `conversation.py` extracts searchable text from each block type. The search bar is a docked `Horizontal` (`#search-bar-row`) that is hidden by default and shown on `/`.

**Error handling:** `parse_file()` and `session_summary()` catch `PermissionError`/`OSError` and return an `error` field rather than raising. The browser shows inaccessible files as `[no access]`; the conversation panel shows the error message.

---

## Keybindings

| Key | Scope | Action |
|---|---|---|
| `j`/`k`, arrows | App | Navigate file browser |
| `Enter` | App | Load session (browser focused) or toggle item (tool/result/response block focused) |
| `Space` | Block | Toggle focused collapsible block |
| `y` | ToolCallBlock | Copy `arguments["command"]` to clipboard (no-op if no `command` key) |
| `Tab` | App | Move focus between collapsible blocks in the conversation |
| `t` | App | Toggle thinking blocks |
| `T` | App | Collapse / expand all turns |
| `e` | App | Toggle all collapsible blocks expanded/collapsed |
| `$` | App | Toggle token/cost footers |
| `r` | App | Reload current file (or refresh cron jobs list in cron mode) |
| `C` | App | Toggle cron mode (swap left panel between session browser and cron browser) |
| `q` | App | Quit |
| `/` | ConversationView | Open search bar |
| `Enter` | Search input | Run search; jumps to first match and expands it |
| `n` | ConversationView | Next match |
| `N` | ConversationView | Previous match |
| `Escape` | ConversationView | Close search bar, clear results |
| `]` | ConversationView | Scroll to next turn |
| `[` | ConversationView | Scroll to previous turn |
| `Enter`/`Space` | TurnSeparator | Collapse / expand turn |
| `d` | ConversationView | Toggle debug log section (only when session loaded via cron run) |
| `Escape`/`Backspace` | CronBrowser | Go back from runs list to jobs list |

`/`, `n`, `N`, `Escape` are bound on `ConversationView` — they fire when the conversation panel or any block inside it has focus. While the search `Input` is focused, `n`/`N` type into the field; they become navigation keys once focus moves to a matched block.

---

## Clipboard

`y` on a focused `ToolCallBlock` copies `arguments["command"]` to the clipboard. Implementation:
1. On Wayland (`WAYLAND_DISPLAY` set): shells out to `wl-copy` (from `wl-clipboard`) — required because VTE-based terminals (GNOME Terminal) silently discard OSC 52.
2. Always calls `app.copy_to_clipboard()` afterwards (OSC 52) — works on kitty, foot, wezterm, etc.

If `wl-copy` is missing: `sudo apt install wl-clipboard`.

---

## Dependencies

- `textual` — TUI framework (includes `rich`)
- `pytest`, `pytest-asyncio` — test suite only
- `wl-clipboard` (system package, optional) — needed for clipboard on Wayland
- No other third-party dependencies

---

## Testing

```bash
venv/bin/pytest tests/ -v
```

Tests use `tmp_path` fixture JSONL files and Textual's `app.run_test()` async pilot. No live sessions directory is touched.

---

## Known Constraints / Future Work

- Assistant text is rendered as plain `Static`, not markdown — easy to upgrade but adds overhead for large sessions
- `session_summary()` scans the file independently from `parse_file()`; could be unified for very large session dirs
- Default sessions dir is hardcoded; override with `--dir`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ram-nat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
