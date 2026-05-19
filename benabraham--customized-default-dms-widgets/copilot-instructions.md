## customized-default-dms-widgets

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Custom QML plugins for DankMaterialShell (a Quickshell-based desktop shell). These are modified copies of DMS built-in widgets with specific customizations applied.

**Upstream:** https://github.com/AvengeMedia/DankMaterialShell
**Original widgets path:** `quickshell/Modules/DankBar/Widgets/`

## Development

No build/lint/test commands - QML plugins are loaded directly by DMS at runtime.

**Testing changes:**
1. Edit QML files
2. Reload shell via `quickshell --reload` or restart

## Code Conventions

- JavaScript in QML: no semicolons, arrow functions, template literals, single quotes
- Property bindings for reactive state propagation
- Event-driven updates with debouncing where needed
- Status colors from Theme: `Theme.primary` (running), `Theme.warning` (paused), `Theme.error`

## Architecture

**Plugin structure:** Each plugin has `PluginName.qml` + `plugin.json` metadata.

**Key services:**
- `PluginService` - Plugin data persistence (`loadPluginData`, `savePluginData`)
- `CompositorService` - Window/workspace management
- `Theme` - Theming system (colors, spacing, fonts)
- `DesktopEntries` - Application metadata

**Data flow:**
```
DankBar (parent) → Plugin Widget → Local state + Services → Reactive UI
```

## Upstream Sync

Use `/sync-with-upstream` slash command to sync with upstream DankMaterialShell changes.

Key customizations to preserve are documented in README.md and `.claude/commands/sync-with-upstream.md`.

## Plugin-Specific Notes

- **CustomRunningApps** - Most complex; has scroll switching, middle-click close, context menu, dynamic title width
- **CustomSystemTrayBar** - Regex-based icon sorting via PluginService
- **CustomWorkspaceSwitcher** - Individual app icons (no grouping), click-to-focus
- **Screensaver** - Daemon plugin; multi-stage DDC brightness dimming for OLED burn-in protection (not a bar widget)
- **dockerManager** - Independent plugin with own architecture; see `dockerManager/CLAUDE.md`

---
> Source: [benabraham/customized-default-dms-widgets](https://github.com/benabraham/customized-default-dms-widgets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
