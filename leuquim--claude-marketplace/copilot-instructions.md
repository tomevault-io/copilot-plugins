## claude-marketplace

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal Claude Code plugins marketplace. Plugins are located in `plugins/` - explore the directory to discover available plugins and their components.

## Project Structure

```
claude-marketplace/
├── .claude-plugin/
│   ├── marketplace.json         # Plugin catalog
│   └── plugin.json              # Root metadata
├── plugins/                     # Individual plugins (explore to discover)
│   └── {plugin-name}/
│       ├── .claude-plugin/
│       │   └── plugin.json      # Plugin manifest
│       ├── agents/              # Specialized agents
│       ├── commands/            # Slash commands
│       └── skills/              # Skill modules
└── TODO.md                      # Backlog
```

## Plugin Development

For Claude Code plugin schema and best practices, fetch the official documentation at runtime using the `claude-code-guide` agent or WebFetch from `https://docs.anthropic.com/en/docs/claude-code`.

## Key Patterns

### Skills as Reference
Skills are comprehensive documentation guides (not executable code) teaching tool usage and workflows.

### Vector Search Integration
Uses `code-vector-cli` (optional, requires pre-indexing). Platform-aware: uses WSL on Windows.

Collections: `code_functions`, `code_classes`, `code_files`, `documentation`, `git_history`, `conversations`

## Quality Standards

- Descriptions under 100 characters
- Rationale-based guidance (avoid "MUST", "NEVER")
- Include `allowed-tools` in frontmatter
- Explicit step-by-step workflows

## Versioning

Bump plugin versions in `plugins/{name}/.claude-plugin/plugin.json` when making changes:

- **Patch (x.x.1)**: Bug fixes, optimizations, documentation
- **Minor (x.1.0)**: New features, new agents/skills, non-breaking changes
- **Major (1.0.0)**: Breaking changes, removed features, significant restructuring

Users with `autoUpdates: false` won't receive changes until they manually update or reinstall.

---
> Source: [leuquim/claude-marketplace](https://github.com/leuquim/claude-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
