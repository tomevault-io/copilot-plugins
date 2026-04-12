## claude-session-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Cinnamon desktop extension + Claude Code hook script for tracking multiple Claude Code sessions. Two components communicate via JSON state files in `~/.local/state/claude-sessions/`:

- **`bin/claude-session-tracker`** — Bash script invoked by Claude Code hooks. Manages per-session JSON state files keyed by session ID. Actions: `session-start`, `notification-idle`, `notification-permission`, `prompt-submit`, `tool-active`, `session-end`, `focus`.
- **`extension/claude-sessions@fletch/extension.js`** — Cinnamon extension (GJS/CJS). Monitors the state directory via polling, shows a floating widget in the bottom-right corner with colored dots in a grid (2 columns). Hover shows a tooltip with status icon and project name. Each dot is clickable to focus its terminal window.

## Architecture

State flow: Claude Code hooks → stdin JSON → `claude-session-tracker` → writes `~/.local/state/claude-sessions/<session_id>.json` → extension polls directory every 1s → reads all JSON files → updates widget.

PID tracking: On `session-start`, the hook script walks up the process tree from `$PPID` to find the actual `claude` process (Claude Code spawns hooks via intermediate shells that exit immediately). The PID is stored in the session JSON so the extension can reap stale sessions whose process has exited. If no `claude` process is found in the ancestor chain, falls back to `$PPID`. Additionally, `session-start` cleans up any existing session files that share the same PID (stale sessions from a previous run that didn't receive `session-end`).

Widget positioning: The extension uses `Main.layoutManager.addChrome()` to create a floating widget that stays above all windows, receives click input, but doesn't appear in alt-tab. Positioned bottom-right of the primary monitor, offset 10px from edges and above any bottom panel (height read from `Main.panelManager.panels`). Repositioned on `monitors-changed` and `allocation-changed`.

Window focusing: On `session-start`, the hook captures the window ID via `$WINDOWID` (if set) or `xdotool getactivewindow` (the terminal is focused since the user just launched claude). It also discovers the PTY path by walking the process tree's fd/0 links. The `focus` action uses `wmctrl -i -a` with hex window ID for cross-workspace activation. For Gnome Terminal tab switching, `session-start` reads `_GTK_WINDOW_OBJECT_PATH` from `xprop` to get the D-Bus window path, then queries the `active-tab` state (the tab is active at launch time, so its index is the current value). On `focus`, after raising the window, if a PTY path is available, the hook re-detects the correct tab index by writing a unique marker escape sequence to the PTY title, then scanning tabs to find the one whose `_NET_WM_NAME` matches the marker (handling tab reorder since session-start). If the stored tab index is still correct it's used directly (fast path). The updated tab index is persisted back to the session JSON. If no PTY is available, `gdbus` calls `org.gtk.Actions.Activate active-tab` with the stored tab index directly.

Theme colors: On `session-start`, the hook walks up from `$cwd` looking for `.terminal-theme`, resolves the theme file from `$DOT_PATH/bash/terminal-themes/` (or `~/Dropbox/dotfiles/bash/terminal-themes/`), and extracts `prompt_fill` into `theme_color` in the session JSON. The extension renders each session as a colored dot using this value (fallback: `#cc241d`). Visual states: permission dots have a white border; idle and active dots have no border. Active (busy) dots pulse (opacity cycles 30–120); idle dots that have been idle for more than 3 seconds and permission dots stay at full opacity (recently-idle dots continue pulsing briefly as a visual transition).

Focus tracking: The extension listens to `global.display` `notify::focus-window` to detect which window is active. When the focused window matches a session's `window_id` (and `tab_index` if present), the dot renders a white inner circle. For tab-level focus detection, the extension subscribes to `org.gtk.Actions.Changed` D-Bus signals on each terminal window's object path, so tab switches update the focus dot instantly rather than waiting for the next poll cycle.

## Install & Test

```bash
./install.sh          # symlinks bin + extension, creates state dir, configures Claude hooks
./test.sh             # runs shellcheck + integration tests for the hook script
```

After install: reload Cinnamon (`Alt+F2 → r`) and enable in System Settings → Extensions.

Dev reload: `./dev-reload.sh` sends a D-Bus `ReloadExtension` call. For extensions this re-runs `disable()` + `enable()` but does **not** re-evaluate the JS file — a full Cinnamon restart (`Alt+F2 → r`) is needed to pick up code changes.

Manual test (no build step):
```bash
echo '{"session_id":"test","cwd":"/tmp"}' | ~/.local/bin/claude-session-tracker notification-idle
cat ~/.local/state/claude-sessions/test.json
echo '{"session_id":"test","cwd":"/tmp"}' | ~/.local/bin/claude-session-tracker session-end
```

Note: manual test sessions created from a shell will be reaped by the extension's PID check (the hook script's `$PPID` exits immediately). To test without reaping, write a JSON file directly with a long-lived PID (e.g. `"pid": 1`).

## Dependencies

Runtime: `xdotool`, `jq`, `wmctrl`, `xprop`, `gdbus`, Cinnamon desktop. No build tools or package managers.

## Conventions

- Bash scripts use `set -euo pipefail`. State file writes use atomic tmp+mv pattern with PID-unique tmp files (`${file}.tmp.$$`) to avoid races between concurrent hook invocations.
- Extension uses Cinnamon's CJS (imports.gi/imports.ui), not ES modules or Node.
- The extension hides itself when no sessions exist (count == 0). All sessions are shown (active, idle, permission).
- The extension exports `init(metadata)`, `enable()`, `disable()` per the Cinnamon extension API. Widget is created via `St.BoxLayout` with dots arranged in a grid.
- The extension spawns external commands via `GLib.spawn_async_with_pipes` with argv arrays, never shell string interpolation, to avoid command injection. Data (e.g. session IDs) is passed via stdin, not embedded in command strings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mockdeep)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mockdeep)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
