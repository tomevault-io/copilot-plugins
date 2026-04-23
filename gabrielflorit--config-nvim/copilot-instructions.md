## config-nvim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

This is a LazyVim-based Neovim configuration that follows the standard LazyVim structure:

- `init.lua`: Entry point that bootstraps lazy.nvim and loads the configuration
- `lua/config/`: Core configuration files
  - `lazy.lua`: Lazy.nvim setup and plugin specifications
  - `options.lua`: Vim options and settings
  - `keymaps.lua`: Custom key mappings
  - `autocmds.lua`: Auto commands
- `lua/plugins/`: Custom plugin configurations that override or extend LazyVim defaults

## Key Configuration Details

### Leader Key
- Leader key is set to `,` (comma) instead of the default space
- Local leader is also `,`

### Core Features
- LazyVim base with these extras enabled:
  - `mini-surround` for text objects
  - `prettier` for formatting
  - `typescript` language support
  - `eslint` for linting
- Line numbers are disabled (`number = false`, `relativenumber = false`)
- Snacks animations are disabled
- Uses `tokyonight` colorscheme

### Plugin Management
- Uses lazy.nvim for plugin management
- Custom plugins in `lua/plugins/` override LazyVim defaults
- Plugins are not lazy-loaded by default (`lazy = false`)
- Auto-updates are enabled but notifications are disabled

### Custom Plugins and Overrides
- **Oil.nvim**: File explorer with `-` key binding, custom keymap overrides
- **Disabled plugins**: Some LazyVim plugins are explicitly disabled in `disabled.lua`
- **Performance**: Several default Vim plugins are disabled for better performance

### File Structure Patterns
- Plugin files return a table/array of plugin specifications
- Each plugin spec can include `opts`, `keys`, `dependencies`, `init`, etc.
- Use `enabled = false` to disable unwanted plugins

## Development Commands

Since this is a Neovim configuration (not a software project), there are no build/test commands. Configuration changes take effect when Neovim is restarted or files are sourced.

### Testing Configuration
- Restart Neovim to test changes: `:qa` then reopen
- Source specific files: `:source %` or `:luafile %`
- Check for errors: `:messages` or `:checkhealth`

### Plugin Management
- Install/update plugins: `:Lazy sync`
- Check plugin status: `:Lazy`
- Clean unused plugins: `:Lazy clean`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gabrielflorit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
