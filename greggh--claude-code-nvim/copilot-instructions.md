## claude-code-nvim

> Claude Code Plugin provides seamless integration between the Claude Code AI assistant and Neovim. It enables direct communication with the Claude Code CLI from within the editor, context-aware interactions, and various utilities to enhance AI-assisted development within Neovim.

# Project: Claude Code Plugin

## Overview

Claude Code Plugin provides seamless integration between the Claude Code AI assistant and Neovim. It enables direct communication with the Claude Code CLI from within the editor, context-aware interactions, and various utilities to enhance AI-assisted development within Neovim.

## Essential Commands

- Run Tests: `env -C /home/gregg/Projects/neovim/plugins/claude-code lua tests/run_tests.lua`
- Check Formatting: `env -C /home/gregg/Projects/neovim/plugins/claude-code stylua lua/ -c`
- Format Code: `env -C /home/gregg/Projects/neovim/plugins/claude-code stylua lua/`
- Run Linter: `env -C /home/gregg/Projects/neovim/plugins/claude-code luacheck lua/`
- Build Documentation: `env -C /home/gregg/Projects/neovim/plugins/claude-code mkdocs build`

## Project Structure

- `/lua/claude-code`: Main plugin code
- `/lua/claude-code/cli`: Claude Code CLI integration
- `/lua/claude-code/ui`: UI components for interactions
- `/lua/claude-code/context`: Context management utilities
- `/after/plugin`: Plugin setup and initialization
- `/tests`: Test files for plugin functionality
- `/doc`: Vim help documentation

## Current Focus

- Integrating nvim-toolkit for shared utilities
- Adding hooks-util as git submodule for development workflow
- Enhancing bidirectional communication with Claude Code CLI
- Implementing better context synchronization
- Adding buffer-specific context management

## Multi-Instance Support

The plugin supports running multiple Claude Code instances, one per git repository root:

- Each git repository maintains its own Claude instance
- Works across multiple Neovim tabs with different projects
- Allows working on multiple projects in parallel
- Configurable via `git.multi_instance` option (defaults to `true`)
- Instances remain in their own directory context when switching between tabs
- Buffer names include the git root path for easy identification

Example configuration to disable multi-instance mode:

```lua
require('claude-code').setup({
  git = {
    multi_instance = false  -- Use a single global Claude instance
  }
})
```

## Documentation Links

- Tasks: `/home/gregg/Projects/docs-projects/neovim-ecosystem-docs/tasks/claude-code-tasks.md`
- Project Status: `/home/gregg/Projects/docs-projects/neovim-ecosystem-docs/project-status.md`

---
> Source: [greggh/claude-code.nvim](https://github.com/greggh/claude-code.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
