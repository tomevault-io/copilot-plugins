## claude-wordpress-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin that provides WordPress performance review skills. It enables Claude to perform comprehensive code reviews for WordPress themes, plugins, and custom code with a focus on performance optimization.

## Repository Structure

```
.claude-plugin/           # Plugin configuration
  plugin.json             # Plugin metadata (name, version, author)
  marketplace.json        # Marketplace registration

skills/                   # Skill definitions
  wp-performance-review/
    SKILL.md              # Main skill file with YAML frontmatter
    references/           # Supporting documentation
      anti-patterns.md    # PHP code anti-patterns reference
      wp-query-guide.md   # WP_Query optimization reference
      caching-guide.md    # Caching strategies reference
      measurement-guide.md # Performance measurement reference

commands/                 # Slash command definitions
  wp-perf-review.md       # Full performance review command
  wp-perf.md              # Quick triage scan command
```

## Adding New Skills

1. Create directory: `skills/wp-your-skill/`
2. Create `SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   name: skill-name
   description: Trigger phrases and when to use. Max 1024 chars.
   ---
   ```
3. Add references in `skills/wp-your-skill/references/` if needed
4. Update `marketplace.json` to include the new skill
5. Update `README.md` skill table

## Adding Slash Commands

Create a markdown file in `commands/` with:
```yaml
---
description: What the command does
argument-hint: [optional-args]
---
```

## Code Standards

PHP examples must follow WordPress PHP Coding Standards:
- Spaces inside parentheses: `function_name( $arg )`
- Use `array()` not `[]`
- Yoda conditions: `if ( true === $value )`
- Snake_case for variables/functions
- Prefix custom functions: `prefix_function_name()`

Use consistent severity labels in skill content:
- **CRITICAL**: Will cause failures at scale (OOM, 500 errors)
- **WARNING**: Degrades performance under load
- **INFO**: Optimization opportunity

## Testing Changes

```bash
# Copy skill to Claude skills directory for testing
cp -r skills/your-skill ~/.claude/skills/

# Restart Claude Code to load changes
```

## Versioning

Update version in both files when releasing:
- `.claude-plugin/plugin.json` - `version` field
- `.claude-plugin/marketplace.json` - `metadata.version` field

---
> Source: [elvismdev/claude-wordpress-skills](https://github.com/elvismdev/claude-wordpress-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
