## lazytilt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
go build -o lazytilt .          # build binary
go build ./...                  # check compilation
./lazytilt                      # run (expects Tilt at localhost:10350)
./lazytilt --host host:port     # custom Tilt API address
```

No tests yet. No linter configured.

## Architecture

lazytilt is a terminal UI for [Tilt.dev](https://tilt.dev), inspired by lazygit. Built with [awesome-gocui](https://github.com/awesome-gocui/gocui) v1.1.0.

### Package structure

- `main.go` - CLI entry point, parses `--host` and `--version` flags
- `internal/app` - Bootstrap: creates tilt client and GUI, wires them together
- `internal/tilt` - Tilt API client. Single `FetchAll()` call to `/api/view` returns resources, session, and logs. Also `TriggerResource()` for builds/restarts.
- `internal/gui` - All TUI code:
  - `gui.go` - Central `Gui` struct, poll loop (2s), render functions for each panel, helpers
  - `layout.go` - Accordion layout: 3 stacked side panels (Status, Resources, Builds) + Logs + Command log + options bar
  - `keybindings.go` - All input handling: panel navigation, resource list nav, log scrolling, search, actions (build/restart/reload), detail view toggle, popups
  - `style.go` - ANSI color helpers, status icons, fuzzy match

### Key patterns

- **Accordion layout**: Focused side panel expands, others collapse to title bar (2 rows). Like lazygit's `ExpandFocusedSidePanel`.
- **Popup focus tracking**: `popupView` field prevents `layout()` from stealing focus via `SetCurrentView`. Popups use `SetViewOnTop` + dynamic keybinding registration/cleanup.
- **Fuzzy search**: `/` opens search bar with custom `searchEditor` that handles char-by-char input. Filters resource list in real-time. Arrow keys/Tab/Enter exit search and keep filter.
- **Detail view**: Enter toggles main panel between logs and resource details (no popup, renders in-place).
- **Navigation**: Tab/Shift+Tab cycle side panels, right arrow/l enters logs, left/h/Esc returns, 0-3 jump to specific panels.
- **Filtered resources**: `filteredResourcesLocked()` returns search-filtered list. All panels (resources, builds, logs) use filtered results.

---
> Source: [tdi/lazytilt](https://github.com/tdi/lazytilt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
