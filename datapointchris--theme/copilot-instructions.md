## theme

> Unified theme generation system that creates consistent color configurations

# Theme System - Claude Code Context

## Overview

Unified theme generation system that creates consistent color configurations
across terminal and desktop applications from a single `theme.yml` source file.
Supports Ghostty, Kitty, tmux, btop, JankyBorders, Hyprland, Waybar, Rofi,
Dunst, Windows Terminal, and more. Each theme in `themes/` provides app configs
that match a corresponding Neovim colorscheme.

## Directory Structure

```text
apps/common/theme/
├── bin/theme           # Theme CLI tool (apply, preview, like/dislike)
├── demo/               # Sample code files for theme preview
├── lib/                # Core libraries and generators
│   ├── generators/     # App-specific generators
│   │   ├── ghostty.sh, kitty.sh     # Terminal emulators (colors)
│   │   ├── ghostty-css.sh           # Ghostty tab custom CSS
│   │   ├── tmux.sh, btop.sh         # Terminal apps
│   │   ├── borders.sh               # JankyBorders (macOS)
│   │   ├── background.sh            # Themed backgrounds (macOS)
│   │   ├── hyprland.sh, hyprlock.sh # Hyprland WM (Arch)
│   │   ├── waybar.sh, rofi.sh       # Desktop apps (Arch)
│   │   ├── dunst.sh, mako.sh        # Notification daemons
│   │   ├── windows-terminal.sh      # WSL terminal
│   │   └── preview.sh               # Theme preview images
│   ├── neovim_generator.py  # Generates Neovim colorscheme plugin
│   └── theme.sh        # Loads theme.yml into shell variables
├── themes/             # 40+ themes with theme.yml source and generated configs
│   ├── gruvbox-dark-hard/  # With generated neovim/
│   ├── rose-pine-darker/   # With generated neovim/
│   ├── kanagawa/           # Terminal configs only (uses plugin for Neovim)
│   └── .../
├── scripts/            # Migration and utility scripts
├── install.sh          # Installation script
├── analysis/           # Research documentation
└── screenshots/        # Theme preview screenshots

# Data locations (XDG-compliant):
# ~/.local/state/theme/history.jsonl   - Unified history (synced via gist)
# ~/.local/state/theme/current         - Current theme ID
# ~/.local/state/theme/sync-state.json - Sync configuration
```

## Theme Categories

### Generated Themes (neovim_colorscheme_source: "generated")

These themes have generated Neovim colorschemes from theme.yml:

| Directory | display_name | Notes |
|-----------|-------------|-------|
| `gruvbox-dark-hard` | Gruvbox Dark Hard | Ghostty-derived, neutral ANSI |
| `rose-pine-darker` | Rose Pine Darker | Base16-derived, darker background |

### Plugin Themes (neovim_colorscheme_source: "plugin")

These themes provide terminal configs that match original Neovim plugins:

- `gruvbox` - Gruvbox (`gruvbox`) - ellisonleao/gruvbox.nvim
- `rose-pine` - Rose Pine (`rose-pine`) - rose-pine/neovim
- `kanagawa` - Kanagawa (`kanagawa`) - rebelot/kanagawa.nvim
- `nordic` - Nordic (`nordic`) - AlexvZyl/nordic.nvim
- `terafox` - Terafox (`terafox`) - EdenEast/nightfox.nvim
- `oceanic-next` - Oceanic Next (`OceanicNext`) -
  mhartington/oceanic-next
- `github-dark-default` - GitHub Dark Default (`github_dark_default`) -
  projekt0n/github-nvim-theme

## Theme Files

Each theme directory contains app-specific configs generated from `theme.yml`:

```text
themes/{theme-id}/
├── theme.yml           # Source palette (required)
├── ghostty.conf        # Ghostty terminal colors
├── ghostty.css         # Ghostty tab custom CSS
├── kitty.conf          # Kitty terminal
├── tmux.conf           # tmux status bar
├── btop.theme          # btop system monitor
├── bordersrc           # JankyBorders (macOS)
├── hyprland.conf       # Hyprland WM (Arch)
├── waybar.css          # Waybar status bar (Arch)
├── hyprlock.conf       # Hyprlock lock screen (Arch)
├── dunst.conf          # Dunst notifications (Arch)
├── rofi.rasi           # Rofi launcher (Arch)
├── windows-terminal.json  # Windows Terminal (WSL)
└── neovim/             # Only for generated themes - colorscheme plugin
```

### theme.yml Format

```yaml
meta:
  id: "gruvbox-dark-hard"              # Directory name, lowercase-hyphen
  display_name: "Gruvbox Dark Hard"    # Pretty name for UI
  neovim_colorscheme_name: "gruvbox-dark-hard"  # What :colorscheme uses
  neovim_colorscheme_source: "generated"  # "generated" or "plugin"
  plugin: null                         # "author/repo" or null
  derived_from: "ghostty-builtin"      # Where colors came from
  variant: "dark"
  author: "morhetz"

base16:
  base00: "#1d2021"  # Background through base0F
  # ...

ansi:
  black: "#..."      # 16 ANSI terminal colors
  # ...

special:
  background: "#..."
  foreground: "#..."
  cursor: "#..."
  # ...

extended:
  # Theme-specific extra colors (optional)
```

## Theme Workflow

### Using Existing Themes

```bash
theme list                       # List with display names
theme change                     # Interactive picker
theme apply gruvbox-dark-hard    # Apply by id
theme current                    # Show current theme
theme like "great contrast"      # Rate current theme
theme reject "too bright"        # Remove from rotation
theme upgrade                    # Update to latest version

# Background management
theme background                 # Show background usage
theme background current         # Show current background
theme background rotate          # Rotate to new background
theme background mode set recolor generated:plasma  # Set modes
theme background source add ~/Pictures/wallpapers   # Add source

# Opacity
theme opacity                    # Show opacity usage
theme opacity current            # Show current opacity
theme opacity set 90             # Set opacity to 90%

# Sync
theme sync                       # Show sync usage
theme sync init                  # Initialize GitHub Gist sync
theme sync status                # Show sync status
```

### Creating a New Theme

1. Create `themes/{id}/theme.yml` with meta, base16, ansi, and special sections
2. Set `neovim_colorscheme_source: "plugin"` if using existing Neovim plugin
3. Set `neovim_colorscheme_source: "generated"` if generating Neovim colorscheme
4. Generate app configs using generators in `lib/generators/`:

```bash
cd apps/common/theme

# Core apps (all platforms)
bash lib/generators/ghostty.sh themes/{id}/theme.yml themes/{id}/ghostty.conf
bash lib/generators/ghostty-css.sh themes/{id}/theme.yml themes/{id}/ghostty.css
bash lib/generators/kitty.sh themes/{id}/theme.yml themes/{id}/kitty.conf
bash lib/generators/tmux.sh themes/{id}/theme.yml themes/{id}/tmux.conf
bash lib/generators/btop.sh themes/{id}/theme.yml themes/{id}/btop.theme

# macOS
bash lib/generators/borders.sh themes/{id}/theme.yml themes/{id}/bordersrc

# Arch/Hyprland
bash lib/generators/hyprland.sh themes/{id}/theme.yml themes/{id}/hyprland.conf
bash lib/generators/waybar.sh themes/{id}/theme.yml themes/{id}/waybar.css
bash lib/generators/dunst.sh themes/{id}/theme.yml themes/{id}/dunst.conf
bash lib/generators/rofi.sh themes/{id}/theme.yml themes/{id}/rofi.rasi

# WSL
bash lib/generators/windows-terminal.sh themes/{id}/theme.yml themes/{id}/windows-terminal.json
```

### Creating a Generated Neovim Colorscheme

If creating a new colorscheme (not using a plugin):

1. Generate Neovim colorscheme:

   ```bash
   uv run --with pyyaml python3 \
     ~/dotfiles/apps/common/theme/lib/neovim_generator.py \
     ~/dotfiles/apps/common/theme/themes/{id}
   ```

2. The colorscheme is auto-loaded by `colorscheme-manager.lua` which scans
   for `neovim/` directories

## Key Insights

- **Same palette ≠ same result**: Hand-crafted Neovim plugins often look
  better than generated colorschemes
- **Generated themes are rare**: Most themes work best with original Neovim
  plugin + generated terminal configs
- **neovim_colorscheme_name may differ from id**: e.g., `oceanic-next`
  directory uses `OceanicNext` colorscheme

## Neovim Integration

The `colorscheme-manager.lua` plugin:

- Dynamically loads generated colorschemes from `themes/*/neovim/` directories
- Builds display name mapping from theme.yml meta fields
- Filters rejected themes from the picker
- Watches `~/.local/state/theme/current` for changes (auto-updates when
  `theme apply` runs)

## Files Reference

| File | Purpose |
|------|---------|
| `bin/theme` | Theme CLI (apply, list, like/dislike, reject, sync) |
| `lib/lib.sh` | Core functions (get_theme_display_info, apply_theme_to_apps) |
| `lib/storage.sh` | Unified JSONL history with machine context |
| `lib/sync.sh` | GitHub Gist synchronization |
| `lib/theme.sh` | Loads theme.yml into shell variables for generators |
| `lib/neovim_generator.py` | Generates Neovim colorscheme from theme.yml |
| `install.sh` | Installation script for fresh installs |
| `scripts/migrate-history.sh` | One-time migration from old format |
| `lib/generators/ghostty.sh` | Ghostty terminal colors |
| `lib/generators/ghostty-css.sh` | Ghostty tab custom CSS |
| `lib/generators/kitty.sh` | Kitty terminal colors |
| `lib/generators/tmux.sh` | tmux status bar theme |
| `lib/generators/btop.sh` | btop system monitor theme |
| `lib/generators/borders.sh` | JankyBorders window highlights (macOS) |
| `lib/generators/background.sh` | Themed background generator (macOS) |
| `lib/generators/hyprland.sh` | Hyprland WM colors (Arch) |
| `lib/generators/waybar.sh` | Waybar status bar (Arch) |
| `lib/generators/rofi.sh` | Rofi launcher (Arch) |
| `lib/generators/dunst.sh` | Dunst notifications (Arch) |
| `lib/generators/windows-terminal.sh` | Windows Terminal (WSL) |
| `lib/generators/preview.sh` | Theme preview image generator |

---
> Source: [datapointchris/theme](https://github.com/datapointchris/theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
