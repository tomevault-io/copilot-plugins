## tmux-agentline

> A tmux plugin that shows real-time AI agent status in the tmux status bar and window list.

# tmux-agentline

A tmux plugin that shows real-time AI agent status in the tmux status bar and window list.

## Architecture

### Claude Code
```
Claude Code Hooks → tmux-agentline hook → State Files → tmux-agentline status → tmux status line
```

### iTerm2
```
Claude Code Hooks → tmux-agentline hook → State Files → claude-status.py → iTerm2 (status bar + tab colors)
```

**State files** are stored in `~/.claude/tmux-stat/<pane_id>.state` as JSON with status, timestamp, session_id, tmux_window, and message fields.

## Key Files

| File | Purpose |
|------|---------|
| `cmd/tmux-agentline/main.go` | Binary entry point with subcommands: hook, status, window, cleanup, json-status, version, setup |
| `internal/setup/setup.go` | Setup subcommand: checks hook config in settings.json, prints setup instructions |
| `internal/hook/hook.go` | Hook subcommand: processes Claude Code hook events, writes state files |
| `internal/status/status.go` | Status subcommand: outputs aggregated status for status bar |
| `internal/window/window.go` | Window subcommand: outputs per-window status for window list |
| `internal/state/state.go` | State file I/O and cleanup |
| `tmux-agentline.tmux` | Plugin entry point: checks hooks, sets up tmux interpolation |
| `scripts/install-hooks.sh` | CLI for hook setup (delegates to binary) and iTerm2 symlink management |
| `scripts/claude-hook.sh` | Shell fallback for hook processing (used when binary not available) |
| `scripts/status.sh` | Shell fallback for status output |
| `scripts/window-status.sh` | Shell fallback for per-window status |
| `scripts/helpers.sh` | Shared shell functions: state file I/O, tmux option retrieval, cleanup |
| `scripts/iterm2/claude-status.py` | iTerm2 status bar component + per-tab coloring via Python API |

## States

- **running** (yellow) - Agent is actively working
- **attention** (red) - Agent needs user input
- **done** (green) - Agent finished or idle

Priority: attention > running > done

## Claude Code Hook Events

The hook handler reads `hook_event_name` from JSON input:
- `SessionStart` → running
- `PreToolUse` / `PostToolUse` → running
- `PermissionRequest` → attention
- `Notification` with `notification_type: permission_prompt` → attention
- `Notification` with `notification_type: idle_prompt` → done
- `Stop` → done
- `SessionEnd` → removes state file

## Hook Installation

`install-hooks.sh` does **not** modify `~/.claude/settings.json` directly. It delegates to `bin/tmux-agentline setup` which:
1. Reads settings.json (read-only) to check if hooks are already present
2. If configured: prints "hooks already configured"
3. If not: prints a copyable `claude` command for the user to run

## Testing

```bash
go test ./...          # Go tests (including setup tests)
./tests/run_tests.sh   # Shell tests
```

Tests use `STATE_DIR` and `TMUX_STAT_SKIP_PANE_CHECK` env vars to isolate from real state.

---
> Source: [nathan-gage/tmux-agentline](https://github.com/nathan-gage/tmux-agentline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
