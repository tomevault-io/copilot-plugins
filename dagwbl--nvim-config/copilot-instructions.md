## nvim-config

> This file serves as a mandate and guide for Gemini CLI to maintain, update, and improve this LazyVim-based Neovim configuration.

# Gemini CLI - Neovim Configuration Management

This file serves as a mandate and guide for Gemini CLI to maintain, update, and improve this LazyVim-based Neovim configuration.

## Project Overview
- **Base Framework:** [LazyVim](https://www.lazyvim.org/)
- **Primary Focus:** Academic writing (Markdown, Quarto, Zotero), personal knowledge management (Obsidian), and efficient modal editing.
- **Platform:** Windows (Cross-device sync across R7000 and WVLIU-GD15).

## Core Principles & Constraints
- **Modal Efficiency:** Prioritize motions and operators over mouse/arrow keys.
- **Surgical Updates:** When modifying `lua/plugins/*.lua`, ensure changes are targeted and don't break existing LazyVim overrides.
- **Host-Specific Logic:** Respect `vim.g.my_paths` defined in `lua/config/options.lua`. Never hardcode paths that should be host-dependent.
- **Writing Workflow:** Maintain integration between Quarto, Zotero, and Obsidian.
- **Keymap Style:** Prefer `<leader>z` for custom system/tooling shortcuts. Use `jk`/`jj` for `<Esc>`.

## Directory Structure
- `lua/config/`: Core settings (`options.lua`, `keymaps.lua`, `autocmds.lua`).
- `lua/plugins/`: Individual plugin specifications. Each file should focus on a specific tool or category (e.g., `obsidian.lua`, `quarto.lua`).
- `lua/utils/`: Custom Lua utility modules.
- `snippets/`: Custom completion snippets.

## Maintenance Guidelines for Gemini
1. **Adding Plugins:** Create a new file in `lua/plugins/` or add to `lua/plugins/custom.lua`. Follow the LazyVim spec format.
2. **Modifying Keymaps:** Always check `lua/config/keymaps.lua` first to avoid collisions.
3. **Updating Options:** Use `lua/config/options.lua` for global settings.
4. **Validation:** After any change, suggest the user restart Neovim or run `:Lazy check` to verify.

## Current Requirements & Goals
- [x] Implement `norm` command improvements (`<leader>zm` added).
- [ ] Improve citation workflow between Zotero and Quarto.
- [ ] Ensure `avante.nvim` is correctly configured for the local environment.
- [ ] Optimize `blink.cmp` for writing-heavy buffers.

## User Preferences
- **Theme:** Catppuccin (with toggle support).
- **Font:** Iosevka Nerd Font / JetBrainsMono.
- **UI:** No arrow keys, centered cursor, blink-off/on cursor in insert mode.
- **Custom Keymaps:**
  - `<leader>zm`: Apply normal mode command (works in Normal and Visual mode).
  - `jk`/`jj`: Escape to normal mode.
  - `;;`: Execute one normal mode command from insert mode.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dagwbl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Dagwbl)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
