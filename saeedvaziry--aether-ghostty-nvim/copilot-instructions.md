## aether-ghostty-nvim

> **Type:** Neovim Colorscheme Plugin

# Aether.nvim - Project Guide for Claude

## Project Overview

**Type:** Neovim Colorscheme Plugin
**Language:** Lua
**Author:** Bjarne Øverli
**License:** MIT

Aether.nvim is a modern, modular Neovim colorscheme with extensive plugin support and base16 color injection capabilities. Inspired by TokyoNight, it provides comprehensive customization through a well-structured architecture.

## Core Architecture

### Entry Points
- `lua/aether/init.lua` - Main module exports `M.setup(opts)` and `M.load(opts)`
- `colors/aether.vim` - Traditional Vim colorscheme entry point (just calls Lua)

### Module Flow
```
init.lua (setup/load)
  ↓
config.lua (merge options with defaults)
  ↓
theme.lua (orchestrate theme application)
  ↓
colors/init.lua (generate color palette)
  ↓
groups/init.lua (load and merge highlight groups)
  ↓
Apply to Neovim via vim.api.nvim_set_hl()
```

## Key Modules

### config.lua
- Defines `M.defaults` with all user-configurable options
- Options: transparent, terminal_colors, styles, sidebars, floats, dim_inactive, lualine_bold, colors, on_colors(), on_highlights(), cache, plugins
- Uses metatable pattern for default fallback

### colors/init.lua (236 lines)
- Defines 40+ color palette (bg, fg, blue, green, red, yellow, etc.)
- Base16 mapping: base00-base0F → semantic colors
- Color derivation: git colors, diff colors, blended backgrounds
- `M.setup(opts)`: Apply base16 injection, calculate derived colors, call on_colors()

### theme.lua
- `M.setup(opts)`: Main orchestration function
  1. Extend config with options
  2. Generate color palette
  3. Load highlight groups
  4. Clear existing highlights if switching
  5. Set termguicolors and colors_name
  6. Apply all highlights
  7. Configure terminal colors

### groups/init.lua
- Plugin registry mapping plugin names to group modules
- `M.setup(colors, opts)`: Load groups based on plugin detection
- Always loads: base.lua, treesitter.lua
- Conditional loads: 23 plugin-specific group files
- Plugin detection modes: `all`, `auto` (checks package.loaded), or explicit

### utils.lua
- Color math: `rgb()`, `blend()`, `darken()`, `lighten()`
- `resolve()`: Flatten style tables into highlight definitions
- `hi()`: Build and execute highlight commands

### hotreload.lua (216 lines) - IMPORTANT
- Auto-reloads theme during development when aether is active
- Listens to BufWritePost on lua files and LazyReload event
- `AetherReload` user command for manual reload
- Smart module clearing with config preservation options
- 100ms defer for optimization

## Highlight Groups Structure

**25 group files totaling 1026 lines:**

Core:
- `base.lua` (162 lines) - Neovim standard groups
- `treesitter.lua` (105 lines) - TreeSitter @ captures
- `lsp.lua` (51 lines) - LSP diagnostics/semantic tokens

Plugins (22 files):
- Navigation: telescope, nvim-tree, neo-tree, trouble, flash, which-key
- Git: gitsigns, diffview, git
- UI: noice, snacks, fidget, indent-blankline
- Dev tools: blink, mason, dap, conform, lint, comment, mini
- Language: markdown

Each implements: `function M.get(c, opts) return { GroupName = {...} } end`

## Code Conventions

### Module Pattern
```lua
local M = {}

---@param c table Color palette
---@param opts table User options
---@return table Highlight group definitions
function M.get(c, opts)
  return {
    GroupName = { fg = c.color, bg = c.bg, bold = true },
    LinkedGroup = "BaseGroup",  -- Link syntax
  }
end

return M
```

### Naming
- Module functions: `M.get()`, `M.setup()`
- Colors: lowercase (bg, fg, blue, red)
- Highlight groups: CamelCase (Normal, Comment, TelescopeNormal)

### Configuration Customization
```lua
-- Base16 injection
opts.colors = { base08 = "#ff0000" }

-- Semantic color adjustment
on_colors = function(colors)
  colors.blue = "#0000ff"
end

-- Specific highlight tweaking
on_highlights = function(hl, c)
  hl.Comment = { fg = c.comment, italic = true }
end
```

## Important Patterns

### Adding Plugin Support
1. Create `lua/aether/groups/plugin-name.lua`
2. Implement `M.get(c, opts)` returning highlight table
3. Register in `groups/init.lua` plugin registry
4. Test with `opts.plugins = { all = true }`

### Color Derivation
- Base16 colors → semantic colors (base08 → red)
- Derived colors calculated in `colors/init.lua`
- Use `utils.blend()` for alpha blending
- Git/diff colors: green/red/yellow base with 25% alpha blends

### Plugin Detection
- `opts.plugins.all = true` → Load all supported
- `opts.plugins.auto = true` → Check package.loaded (default)
- `opts.plugins["plugin-name"] = true` → Explicit enable

## Development Workflow

1. Edit files in `lua/aether/`
2. Plugin auto-reloads on save (if aether is active)
3. Use `:AetherReload` for manual reload
4. No build process needed (pure Lua)
5. Test with multiple plugins enabled/disabled

## File Reference Quick Guide

**Must read for understanding:**
- `lua/aether/init.lua` - Entry point (18 lines)
- `lua/aether/theme.lua` - Orchestration (62 lines)
- `lua/aether/colors/init.lua` - Color system (236 lines)
- `lua/aether/groups/init.lua` - Plugin system (74 lines)

**Common edit targets:**
- `lua/aether/config.lua` - Add/modify options
- `lua/aether/colors/init.lua` - Color palette changes
- `lua/aether/groups/base.lua` - Core highlight groups
- `lua/aether/groups/[plugin].lua` - Plugin-specific groups

**Utilities:**
- `lua/aether/utils.lua` - Color math helpers
- `lua/aether/hotreload.lua` - Development reload logic
- `lua/lualine/themes/aether.lua` - Statusline theme

## Recent Development Focus

Based on git history, recent work includes:
- Theme switching with immediate reload
- Hotreload optimization (100ms defer, conditional reload)
- Module organization improvements
- LuaLS annotations for better IDE support

## Important Notes

1. **Never edit files outside lua/aether/** except lualine theme
2. **Always preserve exact indentation** when editing
3. **Use blend functions** from utils.lua for color mixing
4. **Test with hotreload enabled** during development
5. **Follow existing group file patterns** when adding plugins
6. **Base16 injection affects derived colors** - understand cascade
7. **Plugin auto-detection requires lazy.nvim** for package.loaded
8. **Color names are semantic** - blue doesn't mean #0000ff, check palette
9. **Highlights can link** - use string value instead of table
10. **Terminal colors optional** - controlled by config.terminal_colors

## Testing Checklist

When making changes:
- [ ] Test with transparent = true/false
- [ ] Test with dim_inactive enabled
- [ ] Test plugin detection (all/auto/manual)
- [ ] Verify hotreload works
- [ ] Check lualine theme consistency
- [ ] Test base16 injection
- [ ] Verify terminal colors if enabled
- [ ] Check dark/transparent/normal sidebar/float modes

## Common Tasks

**Add new color:**
1. Edit `lua/aether/colors/init.lua` default_palette
2. Use in group files via `c.your_color`

**Add new plugin support:**
1. Create `lua/aether/groups/plugin-name.lua`
2. Add registry entry in `lua/aether/groups/init.lua`
3. Implement `M.get(c, opts)` with highlights

**Modify existing highlights:**
1. Find group file in `lua/aether/groups/`
2. Edit highlight definition in `M.get()` return table
3. Save and hotreload applies changes

**Debug color issues:**
1. Check palette in `lua/aether/colors/init.lua`
2. Verify base16 mapping
3. Check on_colors() callback if used
4. Inspect final colors with `:lua =require('aether.colorscheme').get()`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saeedvaziry)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/saeedvaziry)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
