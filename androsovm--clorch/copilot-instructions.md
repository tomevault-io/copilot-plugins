## clorch

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Clorch

Mission control TUI dashboard for multiple Claude Code sessions. Tracks agent states via Claude Code hooks (shell scripts that write JSON to `/tmp/clorch/state/`), displays them in a Textual-based terminal UI, and allows approving/denying permissions without leaving the dashboard. No terminal scraping or API calls — pure file-based IPC.

## Build & Development

```bash
python -m venv .venv && source .venv/bin/activate
pip install -e '.[rich]'           # editable install with rich extras
pytest                             # run all tests
pytest tests/test_rules.py         # single test file
pytest tests/test_rules.py -k test_deny_overrides_yolo  # single test
ruff check src/ tests/             # lint (ruff config in pyproject.toml)
ruff format src/ tests/            # format
```

Entry point: `clorch` CLI → `src/clorch/cli.py:main()`. Package layout uses `src/` layout with hatchling build backend.

## Architecture

### Data Flow

```
Claude Code hooks (settings.json)
  → event_handler.sh / notify_handler.sh  (bash, uses jq)
    → /tmp/clorch/state/<session_id>.json  (atomic writes via mktemp+mv)
      → StateManager.scan() reads JSON files every 500ms
        → OrchestratorApp polls and updates TUI widgets
```

### Key Modules

- **`hooks/`** — Shell scripts (`event_handler.sh`, `notify_handler.sh`) that Claude Code calls on every event. `installer.py` merges hook definitions into `~/.claude/settings.json` preserving user hooks. The `_ORCH_MARKER = "clorch/hooks/"` substring identifies Clorch hooks for safe merge/uninstall.

- **`state/`** — `models.py` defines `AgentState` (per-session), `StatusSummary` (aggregate), `ActionItem` (pending permission), and `build_action_queue()`. `manager.py` (`StateManager`) scans/queries/cleans state files. `watcher.py` runs a daemon thread for background polling (used outside TUI).

- **`tui/`** — Textual app in `app.py` (`OrchestratorApp`). Widgets are in `tui/widgets/`: `session_list`, `agent_detail`, `header_bar`, `context_footer`, `action_queue`, `event_log`, `telemetry_panel`, `settings_panel`. Styles in `app.tcss`.

- **`terminal/`** — `TerminalBackend` protocol (`backend.py`) with implementations for iTerm2, Ghostty, Terminal.app. Auto-detection in `detect.py` via `TERM_PROGRAM` env var. Backends provide tty-to-tab mapping, tab activation, and `open_tab()` for new terminal tabs.

- **`tmux/`** — `session.py` (`TmuxSession`) manages tmux lifecycle, send-keys, window/pane operations. `navigator.py` maps agents to tmux windows (by explicit mapping → project name → cwd), implements `jump_to_next_attention()`.

- **`rules.py`** — YOLO mode and per-tool auto-approve/deny rules loaded from `~/.config/clorch/rules.yaml`. First matching rule wins. Deny rules force manual review even in YOLO.

- **`constants.py`** — `AgentStatus` enum (WORKING, IDLE, WAITING_PERMISSION, WAITING_ANSWER, ERROR), Nord Aurora color theme, status display symbols, animation constants.

- **`notifications/`** — `sound.py` (macOS `afplay`), `bell.py` (terminal bell), `macos.py` (osascript notifications).

### Agent Status Lifecycle

```
SessionStart → IDLE
UserPromptSubmit → WORKING
PreToolUse → WORKING (tool_count++)
PermissionRequest → WAITING_PERMISSION
Stop → IDLE (preserves WAITING_ANSWER)
Notification → WAITING_ANSWER
SessionEnd → state file deleted
```

Stale WAITING_PERMISSION states are reset to IDLE after 30s (handles denied permissions where Claude Code doesn't fire a Stop event).

### Permission Approval Mechanism

The TUI sends keystrokes (`y`/`n` + Enter) to tmux panes via `tmux send-keys`. Before sending, `StateManager.verify_status()` re-reads the JSON file to prevent misfires. Non-tmux agents get a tab switch with manual approval prompt.

## Conventions

- Python 3.10+, `from __future__ import annotations` in all modules
- Ruff linter: E/F/W/I rules, line length 100
- Dataclasses for models, no Pydantic
- `AgentStatus` is a `str` enum for direct JSON serialization
- Atomic file writes in shell scripts (mktemp + mv)
- Lazy imports in CLI handlers to keep startup fast
- Tests use `tmp_state_dir` and `make_agent_state` fixtures from `conftest.py`
- Dependencies: `textual>=0.85.0`, `pyyaml>=6.0`, optional `rich`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/androsovm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/androsovm)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
