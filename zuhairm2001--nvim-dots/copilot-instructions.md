## nvim-dots

> - Format: `stylua .` (uses .stylua.toml config)

# AGENTS.md

## Build/Lint/Test Commands
- Format: `stylua .` (uses .stylua.toml config)
- No test framework - this is a Neovim config repo
- Lint: Use `stylua --check .` for Lua formatting

## Code Style Guidelines
- **Language**: Lua 5.1+ (Neovim config)
- **Formatting**: 2 spaces, 120 char width, Unix line endings
- **Imports**: Use `require "module"` syntax, prefer double quotes
- **Naming**: snake_case for variables/functions, PascalCase for modules
- **Structure**: Modular configs in `lua/configs/`, plugins in `lua/plugins/`
- **Error Handling**: Use `pcall` for plugin requires, check nil values
- **Comments**: Minimal, code should be self-explanatory
- **Globals**: Avoid, use `vim.g`/`vim.opt` for Neovim settings

---
> Source: [zuhairm2001/nvim-dots](https://github.com/zuhairm2001/nvim-dots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
