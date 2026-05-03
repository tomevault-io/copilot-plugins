## demo-claude-marketplace

> Provides API security and governance tooling with agents for security auditing, commands for vulnerability scanning, and skills for OpenAPI validation and OWASP compliance. Use this plugin for API design reviews, security audits, and standards enforcement.

# Claude Code Private Marketplace Demo

This repository demonstrates how to structure a private plugin marketplace for Claude Code. It contains two example plugins that showcase the plugin architecture and capabilities.

## Overview

Claude Code supports private marketplaces where teams can distribute custom agents, commands, and skills. This demo shows the complete structure needed to build and distribute plugins.

## Example Plugins

### data-toolkit
Adds data engineering capabilities including specialized agents for pipeline development, commands for data operations, and skills for SQL and ETL workflows. Use this plugin when working on data infrastructure, transformations, or analytics pipelines.

### api-guardian
Provides API security and governance tooling with agents for security auditing, commands for vulnerability scanning, and skills for OpenAPI validation and OWASP compliance. Use this plugin for API design reviews, security audits, and standards enforcement.

## Marketplace Structure

```
repo-root/
├── .claude-plugin/
│   └── marketplace.json     # Marketplace manifest — lists all plugins
└── claude-plugins/
    └── plugin-name/
        ├── .claude-plugin/
        │   └── plugin.json  # Plugin metadata (name, description, version)
        ├── agents/          # Agent definitions with YAML frontmatter
        │   └── agent-name.md
        ├── commands/        # Slash commands with YAML frontmatter
        │   └── command-name.md
        └── skills/          # Skills organized by domain
            └── skill-name/
                ├── SKILL.md # Skill definition with YAML frontmatter
                └── *.md     # Supporting documentation
```

### marketplace.json
The root manifest at `.claude-plugin/marketplace.json` declares the marketplace name, owner, and lists all plugins with relative source paths.

### plugin.json
Contains basic metadata: name, description, and version.

### Agents
Markdown files with YAML frontmatter defining specialized AI agents. Frontmatter includes name, description, model, color, permissionMode, and skills. The body describes the agent's role and principles.

### Commands
Markdown files with YAML frontmatter defining slash commands. Frontmatter includes description. The body outlines scope, execution flow, and constraints.

### Skills
Markdown files with YAML frontmatter defining reusable expertise modules. Frontmatter includes name, description, context, user-invocable flag, and allowed-tools. The body describes the expertise area and key principles.

## Development

This marketplace uses `devstack:mrlm` for development and maintenance.

## Usage

To use this private marketplace, point your Claude Code configuration to this repository. See the Claude Code documentation for detailed setup instructions.

---
> Source: [mrlm-xyz/demo-claude-marketplace](https://github.com/mrlm-xyz/demo-claude-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
