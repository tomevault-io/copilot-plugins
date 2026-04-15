## nvim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal Neovim configuration using lazy.nvim as the plugin manager. The configuration is written entirely in Lua.

## Architecture

```
init.lua                    # Entry point, loads config modules
lua/
├── config/
│   ├── lazy.lua            # lazy.nvim bootstrap and plugin spec loading
│   ├── options.lua         # Vim options and performance settings
│   ├── keymaps.lua         # Global keymaps (non-plugin)
│   └── lsp.lua             # LSP keymaps, diagnostics config, attach handlers
├── plugins/
│   ├── init.lua            # Core plugins (snacks.nvim, colorscheme, icons)
│   ├── lsp/
│   │   ├── mason.lua       # Mason + mason-lspconfig (server installations)
│   │   └── lspconfig.lua   # Additional LSP configs (cspell)
│   └── *.lua               # Individual plugin configurations
└── ai-commit/              # Custom plugin for AI-generated commit messages
    ├── init.lua            # Main module
    ├── providers/          # AI provider implementations (claude, openai, copilot)
    ├── git.lua             # Git operations
    └── prompts.lua         # Commit message prompts
```

## Key Configuration Patterns

**Leader keys**: Space (leader), Backslash (localleader)

**Plugin loading**: Plugins are imported from `lua/plugins/` and `lua/plugins/lsp/` directories via lazy.nvim's `import` feature.

**LSP setup flow**:
1. `mason.lua` installs and configures LSP servers via mason-lspconfig handlers
2. `lspconfig.lua` adds non-Mason LSP configs (like cspell)
3. `config/lsp.lua` sets up keymaps via `LspAttach` autocmd and configures diagnostics

**Snacks.nvim** is the primary UI framework, providing: picker (file finder, grep, LSP navigation), explorer, lazygit integration, terminal, scratch buffers.

## LSP Servers

Managed by Mason: vtsls (TypeScript), lua_ls, tailwindcss, html, cssls, eslint, jsonls

Manual: cspell (spelling)

## Formatting & Linting

- **conform.nvim**: Format on save using prettier (JS/TS/CSS/HTML/JSON/YAML/MD) and stylua (Lua)
- **ESLint LSP**: Used instead of nvim-lint for JavaScript/TypeScript linting
- Format keymap: `<leader>f`

## AI Commit Messages

The `ai-commit` module generates commit messages. Usage:
1. Stage changes
2. Open commit buffer (`:G commit`)
3. Run `:GenerateCommitMessage` or `<leader>gm`

Supports claude, openai, and copilot providers. Configure in `lua/plugins/ai-commit.lua`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/antoniowd)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/antoniowd)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
