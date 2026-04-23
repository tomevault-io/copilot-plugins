## 8b-is-mp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A Claude Code plugin marketplace distributing the `compound-engineering` plugin. Philosophy: **each unit of work makes subsequent work easier.**

## Verification Commands

```bash
# Count components (agents are in subdirectories)
find plugins/compound-engineering/agents -name "*.md" | wc -l
ls plugins/compound-engineering/commands/*.md plugins/compound-engineering/commands/*/*.md 2>/dev/null | wc -l
ls -d plugins/compound-engineering/skills/*/ 2>/dev/null | wc -l

# Validate JSON
cat .claude-plugin/marketplace.json | jq .
cat plugins/compound-engineering/.claude-plugin/plugin.json | jq .
```

## Updating the Plugin

When adding/removing agents, commands, or skills:

1. **Count actual files** using commands above
2. **Update descriptions** in all three places (must match):
   - `plugins/compound-engineering/.claude-plugin/plugin.json` → `description`
   - `.claude-plugin/marketplace.json` → plugin `description`
   - `plugins/compound-engineering/README.md` → component table
3. **Bump version** in both JSON files (semver: major.minor.patch)
4. **Update CHANGELOG.md**
5. **Rebuild docs**: `claude /release-docs`

## Adding Components

### New Agent

1. Create file in appropriate category: `plugins/compound-engineering/agents/{category}/name.md`
2. Categories: `review/`, `research/`, `design/`, `workflow/`, `docs/`
3. Run verification commands and update counts

### New Command

1. Create `plugins/compound-engineering/commands/command-name.md`
2. For workflow commands, use `commands/workflows/` and prefix name with `workflows:`
3. Run verification commands and update counts

### New Skill

Create directory with SKILL.md:
```
plugins/compound-engineering/skills/skill-name/
├── SKILL.md           # Required: has YAML frontmatter (name, description)
├── references/        # Optional: reference docs
└── scripts/           # Optional: helper scripts
```

SKILL.md format:
```markdown
---
name: skill-name
description: This skill should be used when... (third person)
---
```

## Testing

```bash
# Install marketplace locally
claude /plugin marketplace add /path/to/8b-is-mp

# Install plugin
claude /plugin install 8b-compound-engineering

# Test
claude /workflows:review
claude agent kieran-rails-reviewer "test"
```

## Documentation Site

Static HTML in `/docs` (GitHub Pages). After changes run `claude /release-docs` to regenerate.

## Commit Messages

Format: `[Action] [component]` - e.g., `Add security-sentinel agent`, `Fix skill count mismatch`

## Key Learnings

### Component counts must match everywhere
Counts appear in plugin.json description, marketplace.json description, and README.md. Always use verification commands before updating.

### Stick to official Claude Code spec
Don't add custom fields to marketplace.json or plugin.json. See [Plugin Reference](https://docs.claude.com/en/docs/claude-code/plugins-reference).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/8b-is) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
