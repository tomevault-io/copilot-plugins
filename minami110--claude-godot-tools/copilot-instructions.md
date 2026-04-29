## claude-godot-tools

> This repository contains Claude plugins for Godot Engine development, including:

# Claude Godot Tools - Development Guidelines

## Project Overview

This repository contains Claude plugins for Godot Engine development, including:

- **gdscript-toolkit** - GDScript utilities for file management, validation, and formatting
- **gdunit4-toolkit** - gdUnit4 testing framework integration with test runners, writers, and analyzers

## Plugin Validation

**IMPORTANT**: After modifying any plugin-related files, always validate the marketplace and plugin configurations.

### Required Validation Steps

**Validate the marketplace manifest**:
```bash
claude plugin validate .claude-plugin/marketplace.json
```

This validates the entire marketplace configuration including all plugins, skills, and agents.

### When to Validate

Run validation after:
- Modifying `.claude-plugin/marketplace.json`
- Adding/removing skills or agents
- Updating skill manifests (SKILL.md files)
- Changing plugin metadata or structure
- Modifying agent configurations

This ensures all plugin configurations are properly formatted and compatible with Claude Code before committing changes.

---
> Source: [minami110/claude-godot-tools](https://github.com/minami110/claude-godot-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
