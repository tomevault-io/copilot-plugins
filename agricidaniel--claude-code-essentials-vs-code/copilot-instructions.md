## claude-code-essentials-vs-code

> A comprehensive setup guide and toolkit for Claude Code with VS Code. Includes

# Claude Code Essentials for VS Code

## Project Overview

A comprehensive setup guide and toolkit for Claude Code with VS Code. Includes
development tool installation, VS Code extension management, Claude Code feature
documentation, and starter templates for skills, commands, and agents.

## Architecture

```
claude-code-essentials-vs-code/
  CLAUDE.md                          # Project instructions (this file)
  README.md                          # Main guide with install instructions
  docs/                              # Detailed guides
    dev-setup-guide.md              # Core tools, package managers, CLI
    vscode-extensions-guide.md      # VS Code extensions for Claude Code
    claude-code-guide.md            # Claude Code features and workflows
    claude-code-resources.md        # Curated resource collection
  scripts/                           # Setup automation
    setup.sh                        # Unix setup (tools + extensions)
    setup.ps1                       # Windows setup (tools + extensions)
    install-extensions.sh           # VS Code extension installer (Unix)
    install-extensions.ps1          # VS Code extension installer (Windows)
  templates/                         # Starter templates
    CLAUDE.md                       # Example CLAUDE.md for user projects
    settings.json                   # Recommended VS Code settings
    skills/                         # Skill templates (Tier 1-4)
    commands/                       # Custom command templates
    agents/                         # Agent templates
```

## Usage

```bash
# One-command setup (Unix)
curl -fsSL https://raw.githubusercontent.com/AgriciDaniel/claude-code-essentials-vs-code/main/scripts/setup.sh | bash

# One-command setup (Windows)
irm https://raw.githubusercontent.com/AgriciDaniel/claude-code-essentials-vs-code/main/scripts/setup.ps1 | iex
```

---
> Source: [AgriciDaniel/claude-code-essentials-vs-code](https://github.com/AgriciDaniel/claude-code-essentials-vs-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
