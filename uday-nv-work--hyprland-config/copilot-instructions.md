## hyprland-config

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Hyprland window manager configuration repository based on JaKooLit's Hyprland-Dots. It contains a comprehensive setup with organized configuration files, scripts, and customization options for a modern Linux desktop environment.

## Architecture and Structure

### Core Configuration Files
- **hyprland.conf**: Main entry point that sources all other configuration files
- **monitors.conf**: Display/monitor configuration (managed by nwg-displays)
- **workspaces.conf**: Workspace configuration
- **hypridle.conf**: Idle daemon configuration
- **hyprlock.conf**: Screen lock configuration

### Configuration Organization
The configuration is modularly organized into several directories:

**configs/**: Default system configurations
- `Keybinds.conf`: Core keybindings (SUPER as main modifier)

**UserConfigs/**: User-customizable configurations (preserved during updates)
- `01-UserDefaults.conf`: Default applications and environment variables
- `UserSettings.conf`: Main Hyprland settings
- `UserKeybinds.conf`: Additional user keybindings
- `UserDecorations.conf`: Window decorations and visual settings
- `UserAnimations.conf`: Animation configurations
- `WindowRules.conf`: Window rules and layer rules
- `Startup_Apps.conf`: Applications to launch at startup
- `ENVariables.conf`: Environment variables

**scripts/**: System automation scripts (40+ utilities)
- Window management, screenshot tools, media controls, theming, brightness/volume controls
- All scripts are located in `$HOME/.config/hypr/scripts`

**UserScripts/**: User-added scripts (preserved during updates)
- Custom utilities like weather, wallpaper management, theme switching

### Key Features
- **Modular Configuration**: Each aspect (keybinds, decorations, animations) is in separate files
- **Update-Safe**: UserConfigs and UserScripts directories are preserved during system updates
- **Initial Boot Script**: `initial-boot.sh` runs once to set up theming, wallpapers, and system preferences
- **Monitor Profiles**: Support for multiple display configurations via Monitor_Profiles directory
- **Theme Integration**: Wallust for color scheme generation, GTK/Qt theming integration

### Default Applications
- Terminal: Ghostty (`$term = ghostty`)
- File Manager: Thunar (`$files = thunar`)
- Editor: Neovim (`EDITOR=nvim`)

### Key Navigation
- Main Modifier: `SUPER` key
- Window focus: SUPER + h/j/k/l (vim-style)
- Window movement: SUPER + CTRL + h/j/k/l
- Window resizing: SUPER + SHIFT + h/j/k/l
- Workspace switching: SUPER + [1-9,0]
- Smart launcher: SUPER + D (uses smart-launch.sh)

### Script Variables
Important path variables used throughout the configuration:
- `$scriptsDir = $HOME/.config/hypr/scripts`
- `$UserConfigs = $HOME/.config/hypr/UserConfigs`
- `$UserScripts = $HOME/.config/hypr/UserScripts`

## Development Guidelines

### Making Configuration Changes
1. **User configurations**: Edit files in `UserConfigs/` directory - these are preserved during updates
2. **Custom scripts**: Add new scripts to `UserScripts/` directory
3. **Monitor profiles**: Create display profiles in `Monitor_Profiles/` directory using nwg-displays
4. **Avoid modifying**: Core files in `configs/` and `scripts/` as these may be overwritten during updates

### Key Configuration Files to Customize
- `UserConfigs/01-UserDefaults.conf`: Change default terminal, file manager, editor
- `UserConfigs/UserKeybinds.conf`: Add custom keybindings without conflicting with defaults
- `UserConfigs/Startup_Apps.conf`: Configure autostart applications
- `UserConfigs/WindowRules.conf`: Add window-specific rules

### Understanding Window Rules
The configuration includes specific workspace assignments:
- Workspace 1: Microsoft Teams in Brave browser
- Workspace 2: General Brave browser windows and main terminal
- Workspace 3: Secondary applications and client terminals

### Theming and Wallpapers
- Wallpaper management through wallust and swww
- Current wallpaper stored in `wallpaper_effects/.wallpaper_current`
- Theme colors automatically generated from wallpaper using wallust
- GTK and Qt theming synchronized with Hyprland colors
<!-- dgc-policy-v11 -->
# Dual-Graph Context Policy

This project uses a local dual-graph MCP server for efficient context retrieval.

## MANDATORY: Always follow this order

1. **Call `graph_continue` first** — before any file exploration, grep, or code reading.

2. **If `graph_continue` returns `needs_project=true`**: call `graph_scan` with the
   current project directory (`pwd`). Do NOT ask the user.

3. **If `graph_continue` returns `skip=true`**: project has fewer than 5 files.
   Do NOT do broad or recursive exploration. Read only specific files if their names
   are mentioned, or ask the user what to work on.

4. **Read `recommended_files`** using `graph_read` — **one call per file**.
   - `graph_read` accepts a single `file` parameter (string). Call it separately for each
     recommended file. Do NOT pass an array or batch multiple files into one call.
   - `recommended_files` may contain `file::symbol` entries (e.g. `src/auth.ts::handleLogin`).
     Pass them verbatim to `graph_read(file: "src/auth.ts::handleLogin")` — it reads only
     that symbol's lines, not the full file.
   - Example: if `recommended_files` is `["src/auth.ts::handleLogin", "src/db.ts"]`,
     call `graph_read(file: "src/auth.ts::handleLogin")` and `graph_read(file: "src/db.ts")`
     as two separate calls (they can be parallel).

5. **Check `confidence` and obey the caps strictly:**
   - `confidence=high` -> Stop. Do NOT grep or explore further.
   - `confidence=medium` -> If recommended files are insufficient, call `fallback_rg`
     at most `max_supplementary_greps` time(s) with specific terms, then `graph_read`
     at most `max_supplementary_files` additional file(s). Then stop.
   - `confidence=low` -> Call `fallback_rg` at most `max_supplementary_greps` time(s),
     then `graph_read` at most `max_supplementary_files` file(s). Then stop.

## Token Usage

A `token-counter` MCP is available for tracking live token usage.

- To check how many tokens a large file or text will cost **before** reading it:
  `count_tokens({text: "<content>"})`
- To log actual usage after a task completes (if the user asks):
  `log_usage({input_tokens: <est>, output_tokens: <est>, description: "<task>"})`
- To show the user their running session cost:
  `get_session_stats()`

Live dashboard URL is printed at startup next to "Token usage".

## Rules

- Do NOT use `rg`, `grep`, or bash file exploration before calling `graph_continue`.
- Do NOT do broad/recursive exploration at any confidence level.
- `max_supplementary_greps` and `max_supplementary_files` are hard caps - never exceed them.
- Do NOT dump full chat history.
- Do NOT call `graph_retrieve` more than once per turn.
- After edits, call `graph_register_edit` with the changed files. Use `file::symbol` notation (e.g. `src/auth.ts::handleLogin`) when the edit targets a specific function, class, or hook.

## Context Store

Whenever you make a decision, identify a task, note a next step, fact, or blocker during a conversation, call `graph_add_memory`.

**To add an entry:**
```
graph_add_memory(type="decision|task|next|fact|blocker", content="one sentence max 15 words", tags=["topic"], files=["relevant/file.ts"])
```

**Do NOT write context-store.json directly** — always use `graph_add_memory`. It applies pruning and keeps the store healthy.

**Rules:**
- Only log things worth remembering across sessions (not every minor detail)
- `content` must be under 15 words
- `files` lists the files this decision/task relates to (can be empty)
- Log immediately when the item arises — not at session end

## Session End

When the user signals they are done (e.g. "bye", "done", "wrap up", "end session"), proactively update `CONTEXT.md` in the project root with:
- **Current Task**: one sentence on what was being worked on
- **Key Decisions**: bullet list, max 3 items
- **Next Steps**: bullet list, max 3 items

Keep `CONTEXT.md` under 20 lines total. Do NOT summarize the full conversation — only what's needed to resume next session.

---
> Source: [uday-nv-work/hyprland-config](https://github.com/uday-nv-work/hyprland-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
