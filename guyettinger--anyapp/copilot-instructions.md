## anyapp

> Documentation conventions for markdown files and when to create docs


# Documentation Practices

## Location and Naming

- All project documentation files (`*.md`) go in `docs/`
- Use `SCREAMING_SNAKE_CASE.md` for feature docs (e.g., `SELF_MODIFICATION_SYSTEM.md`)
- Exception: `README.md` stays at repository root
- Exception: `CLAUDE.md` stays at repository root (agent context)

## When to Create Documentation

Create documentation for:

- New features with non-obvious behavior
- Complex workflows spanning multiple files
- Architecture decisions that affect multiple areas
- Learnings that are non-obvious or surprising (what worked, what didn't, why)
- Integration guides for external services (MCP servers, APIs)

## Documentation Structure

```markdown
# Feature Name

## Overview
Brief description of what this feature does.

## Architecture
How the feature is structured across the codebase.

## Usage
How to use the feature with examples.

## Configuration
Any settings or environment variables.

## Troubleshooting
Common issues and solutions.
```

## Code Examples in Docs

- Include working code examples
- Use language-specific code blocks
- Keep examples minimal but complete
- Update examples when code changes

## Don't Document

- Obvious behavior already clear from code
- Implementation details that change frequently
- Information already in TSDoc comments

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/guyettinger)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/guyettinger)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
