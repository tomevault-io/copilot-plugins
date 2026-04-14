## claude-config

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains:

- **Custom Claude Code skills** that extend functionality through structured workflows. Skills are capabilities defined in markdown files that Claude Code invokes to perform specialized tasks like creating Obsidian notes, enhancing text, analyzing AWS documentation, and configuring Zellij terminal multiplexer.
- **User memory file** (`user_memory/CLAUDE.md`) containing user-specific instructions and preferences for Claude Code interactions.

## Repository Structure

```
claude-skills/
├── CLAUDE.md                    # This file
├── README.md                    # Public documentation
├── LICENSE                      # MIT license
├── user_memory/                 # User-specific Claude instructions
│   └── CLAUDE.md                # Personal preferences and guidelines
└── skills/                      # Skills directory
    └── [skill-name]/
        ├── SKILL.md             # Skill definition with frontmatter and workflow
        └── references/          # Optional reference documentation
            └── *.md             # Supporting documentation
```

## Architecture

### Skill Definition Pattern

Each skill follows a consistent structure:

1. **YAML Frontmatter**: Metadata including name, description, license, and version
2. **Workflow Instructions**: Step-by-step execution logic for Claude to follow
3. **Integration Patterns**: MCP server tool usage, user confirmation points
4. **Reference Materials**: Supporting documentation in `references/` subdirectories

The frontmatter `description` field is critical—it defines trigger phrases that activate the skill and explains when Claude should use it.

### MCP Server Integration

Skills integrate with external systems through MCP (Model Context Protocol) servers:

- **mcp-obsidian**: Note creation, search, tag management (`make-note`, `sparring`, `text-enhancer`)
- **aws-documentation**: AWS service documentation lookup (`sparring`, `text-enhancer`)

Skills use these tools to gather context before performing actions, ensuring informed decision-making.

### User Confirmation Pattern

Skills implement user confirmation at critical decision points:

1. Gather context from MCP servers or local files
2. Analyze and prepare suggestions
3. Present options to user for confirmation
4. Execute action only after approval
5. Report results with specific details

This pattern prevents unwanted modifications to external systems like Obsidian vaults.

## Creating New Skills

When creating a new skill:

1. **Directory**: Create lowercase, hyphenated directory (e.g., `my-skill`)
2. **SKILL.md**: Include YAML frontmatter with clear trigger phrases in description
3. **Workflow sections**: Use `##` headers for major steps, `###` for substeps
4. **User interactions**: Explicitly state when to wait for user input
5. **MCP tools**: Document which tools are used and when
6. **Examples**: Include code blocks showing expected formats
7. **References**: Store supporting docs in `references/` subdirectory if needed

### Frontmatter Template

```yaml
---
name: skill-name
description: >
  Brief description of what this skill does. Include trigger phrases like
  "Use when the user asks to..." Explain activation conditions clearly.
license: MIT
metadata:
  version: 1.0
---
```

### Workflow Guidelines

- Use numbered lists for sequential steps
- Include decision points and conditional logic
- Specify exact tool calls with parameters
- Show expected input/output formats in code blocks
- Clarify when to halt execution vs. continue
- Document error handling and edge cases

## Existing Skills

### make-note
Creates structured notes in Obsidian with intelligent tag suggestions based on vault patterns. Scans existing tags, suggests relevant ones, waits for confirmation, then creates note in Resources folder with proper frontmatter.

**Key MCP tools**: `obsidian_get_tags`, `obsidian_append_content`

### sparring
Critical thinking partner for technical concepts and strategy. Researches user's Obsidian notes to understand context, identifies gaps and assumptions, provides constructive challenge. For AWS topics, consults AWS documentation.

**Key MCP tools**: `obsidian_simple_search`, `obsidian_complex_search`, `search_documentation`, `read_documentation`

### text-enhancer
Enhances professional and technical text with grammar correction, clarity improvements, and factual verification. Integrates with AWS Documentation and Obsidian to verify technical accuracy and find relevant context.

**Key MCP tools**: `obsidian_simple_search`, `search_documentation`, `read_documentation`

### verbalized-sampling
Prompt engineering technique to overcome mode collapse in LLM responses by generating multiple answers with probabilities. Useful for creative tasks, brainstorming, and exploring alternative solutions.

**Key MCP tools**: None (pure prompting technique)

### zellij-config
Comprehensive Zellij terminal multiplexer configuration management. Handles config.kdl files, layouts, themes, keybindings, and plugins. Configuration lives in `~/dotfiles/.config/zellij/`.

**Key MCP tools**: None (file manipulation only)

**Important**: Always use `ls -a` when checking for Zellij config files due to hidden directories. Pull from Git before modifying `~/dotfiles/`.

## Development Workflow

This repository is primarily documentation-driven. There are no build commands, tests, or compilation steps.

### Making Changes

1. **Read existing skill**: Understand the current structure and workflow
2. **Edit SKILL.md**: Make changes following the established pattern
3. **Maintain consistency**: Match the style and structure of existing skills
4. **Update README.md**: If adding new skills or changing capabilities

### Version Management

Increment the `metadata.version` field in frontmatter when making significant changes to a skill's workflow or capabilities.

## Git Conventions

Based on recent commits, follow these patterns:

- Use descriptive commit messages (e.g., "Added a new skill: verbalized-sampling")
- Keep commits focused on single logical changes
- Commit new skills as complete units
- Fix formatting issues separately from content changes

## Style Guidelines

### Markdown Formatting

- Use ATX headers (`##`, `###`) consistently
- Code blocks must specify language (e.g., ````yaml`, ````kdl`, ````bash`)
- Use bullet lists for options, numbered lists for sequential steps
- Include blank lines before and after headers, code blocks, and lists

### Writing Style

- Be concise and direct
- Use imperative mood for instructions ("Use the tool", not "You should use")
- Avoid buzzwords and marketing language
- Include specific examples and code snippets
- Document edge cases and error conditions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/glnds)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/glnds)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
