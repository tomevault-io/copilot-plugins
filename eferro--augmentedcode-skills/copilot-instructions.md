## augmentedcode-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a curated collection of **Claude Code Skills** - specialized prompts that give Claude Code domain-specific expertise. Skills are distributed as first-level directories that users install into their `.claude/skills/` (project-level) or `~/.claude/skills/` (personal) directories.

## Architecture

### Skill Directory Structure

Each skill is a **first-level directory** following the [Anthropic standard](https://code.claude.com/docs/en/skills.md):

```
skill-name/
├── SKILL.md        # REQUIRED - Main skill prompt with YAML frontmatter
├── README.md       # Skill documentation and usage guide
├── reference.md    # Optional - Detailed examples and patterns
├── LICENSE.txt     # Optional - License and attribution
└── scripts/        # Optional - Utility scripts
```

### SKILL.md Requirements

Every `SKILL.md` must include YAML frontmatter:

```yaml
---
name: skill-name                    # Lowercase, hyphens only, max 64 chars
description: When Claude should use this skill...  # Max 1024 chars
---
```

The `description` field is critical - Claude uses it to determine when to activate the skill. Include specific keywords and use cases.

### Progressive Disclosure Pattern

Keep `SKILL.md` focused (under 500 lines). Link to supporting files for detailed content:
- `reference.md` for detailed patterns, examples, and API docs
- `README.md` for user-facing documentation and installation instructions
- `scripts/` for utility scripts (saves context tokens)

## Working with Skills

### Adding a New Skill

1. Create a new first-level directory with the skill name (lowercase, hyphens)
2. Create `SKILL.md` with required YAML frontmatter
3. Write clear activation keywords in the `description` field
4. Add `README.md` with installation instructions and usage examples
5. If needed, add `reference.md` for detailed patterns/examples
6. Include proper attribution in `LICENSE.txt` if using external frameworks
7. Update the main `README.md` to list the new skill

### Modifying Existing Skills

When editing skills:
- **SKILL.md**: Keep focused on core instructions. Link to reference files for details
- **reference.md**: Add detailed examples, patterns, code snippets
- **README.md**: Update usage examples and documentation
- Test that file references use correct paths (e.g., `[reference.md](reference.md)`)

### Content Requirements

- All content must be in **English**
- Include proper **attribution** for frameworks/methodologies when using external concepts
- Provide concrete, actionable examples
- Use clear section headings and formatting
- Link between files using relative paths

## Distribution

Users install skills by copying directories:

**Project-level** (shared with team):
```bash
mkdir -p .claude/skills
cp -r path/to/skill-name .claude/skills/
```

**Personal** (across all projects):
```bash
mkdir -p ~/.claude/skills
cp -r path/to/skill-name ~/.claude/skills/
```

Users can invoke skills manually with `/skill-name` or let Claude activate them automatically based on context.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eferro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
