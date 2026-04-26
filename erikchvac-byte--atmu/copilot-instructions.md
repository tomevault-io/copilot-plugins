## atmu

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ATMu (Unmutable Tmux Dashboard) is a locked, immutable 3-pane tmux dashboard system. The core concept is to create a fixed tmux layout that cannot be modified by users during runtime - preventing accidental pane resizing, closing, or layout changes.

## Architecture

### Configuration Hierarchy
The tmux configuration uses a layered approach:
1. `.tmux.conf` - Main configuration file that sources the unmutable config
2. `.tmux.conf.unmutable` - Core locked settings that disable all layout modification hotkeys
3. `.tmuxinator.yml` - Session layout definition (200x59 terminal, 3-pane split)

### Key Components

**Unmutable Configuration System** ([.tmux.conf.unmutable](.tmux.conf.unmutable))
- Disables all resizing hotkeys (Alt+arrows, Ctrl+arrows)
- Prevents pane splitting/closing operations
- Disables mouse interactions
- Sets `remain-on-exit on` to prevent panes from closing when processes exit
- Locks status bar updates with `status-interval 0`
- Prevents window management operations

**Launcher Script** ([launch-dashboard.sh](launch-dashboard.sh))
- Primary entry point for starting the dashboard
- Checks dependencies (tmux, tmuxinator)
- Supports `--no-borders` flag for aesthetic customization
- Handles session attachment if already running
- Falls back to manual tmux session creation if tmuxinator fails
- Uses fixed dimensions: 200x59 terminal size

**Desktop Integration** ([atmudashboard.desktop](atmudashboard.desktop))
- FreeDesktop entry for launching from application menus
- Executes launch-dashboard.sh in a terminal

## Running the Dashboard

```bash
# Standard launch
./launch-dashboard.sh

# Launch without borders
./launch-dashboard.sh --no-borders

# Or via tmuxinator directly
mux start atmudashboard
# OR
tmuxinator start atmudashboard
```

## Development Workflow

When modifying the unmutable configuration:
1. Edit `.tmux.conf.unmutable` for locked behavior changes
2. Test by killing existing session: `tmux kill-session -t atmudashboard`
3. Relaunch with `./launch-dashboard.sh`

When changing layout:
1. Edit `.tmuxinator.yml` to modify pane arrangement
2. The layout string format is: `width,heightxheight,x,y{pane_splits}`

## Dependencies

- **tmux** - Terminal multiplexer
- **tmuxinator** (or `mux` alias) - Session manager
- **bash** - For launch script execution

## Configuration Files

- `.tmux.conf` - Entry point, sources unmutable config
- `.tmux.conf.unmutable` - Core locked settings (should be installed to `~/.tmux.conf.unmutable`)
- `.tmuxinator.yml` - Session definition (should be installed to `~/.tmuxinator/atmudashboard.yml`)
- `atmudashboard.desktop` - Desktop launcher file
- **`ADR.md`** - Architecture Decision Record (READ THIS FIRST for design context)

## Before Working on This Project

1. **READ [ADR.md](ADR.md) FIRST** - Contains all architectural decisions, testing results, and known issues
2. **Update ADR.md** after making any architectural changes or before compacting conversations
3. **Known Issues** are documented in ADR.md - check there before debugging

## Testing

Use the test scripts in the project root:
- `test-dashboard.sh` - Original comprehensive test suite (has variable expansion issues)
- `test-final.sh` - Final validation test

Run tests from within WSL:
```bash
cd ~/Dev/ATMu
./test-final.sh
```

## Known Issues (See ADR.md for details)

1. **Line 65 Syntax Error**: Fixed via `sed -i '65s/unbind-key -n }/unbind-key -n "}"/' ~/.tmux.conf.unmutable`
2. **Manual Config Loading**: Immutability config must be sourced with `tmux source-file ~/.tmux.conf.unmutable`
3. **Dimensions**: Actual pane sizes depend on terminal size, may not match 200x59 target
4. **Test Scripts**: Have bash variable expansion issues when run via `wsl bash -c` (use `wsl bash -l -c` instead)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/erikchvac-byte) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
