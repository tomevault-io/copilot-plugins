## happy-claude-skills

> This repository provides a collection of practical skills for AI coding agents. Skills are packaged as SKILL.md files with accompanying scripts, compatible with any agent that supports the [skills](https://github.com/vercel-labs/skills) ecosystem.

# AGENTS.md

This repository provides a collection of practical skills for AI coding agents. Skills are packaged as SKILL.md files with accompanying scripts, compatible with any agent that supports the [skills](https://github.com/vercel-labs/skills) ecosystem.

## Available Skills

| Skill | Description |
|-------|-------------|
| [1password](skills/1password/) | Manage passwords and API credentials with 1Password CLI (`op`) |
| [browser](skills/browser/) | Browser automation via Chrome DevTools Protocol |
| [docx-format-replicator](skills/docx-format-replicator/) | Extract and replicate Word document formatting |
| [trends-bulletin](skills/trends-bulletin/) | Multi-platform trending topics bulletin (6 platforms → Telegram) |
| [video-processor](skills/video-processor/) | Download, convert, and transcribe videos |
| [wechat-article-writer](skills/wechat-article-writer/) | Automated WeChat article writing workflow |

## Skill Format

Each skill lives in `skills/<name>/` and contains:
- `SKILL.md` — Frontmatter (name, description) + instructions
- `scripts/` — Executable scripts (bash, python, node, bun)

## Installation

```bash
# Install via skills CLI (works with Claude Code, Codex, Droid, OpenClaw, Cursor, and 40+ agents)
npx skills add iamzhihuix/happy-claude-skills

# Install specific skills
npx skills add iamzhihuix/happy-claude-skills --skill browser --skill 1password

# Install to a specific agent
npx skills add iamzhihuix/happy-claude-skills --agent codex
npx skills add iamzhihuix/happy-claude-skills --agent droid
npx skills add iamzhihuix/happy-claude-skills --agent openclaw
```

## Contributing

When creating or modifying skills:
1. Each skill must have a `SKILL.md` with YAML frontmatter containing `name` and `description`
2. Scripts should be executable and self-contained
3. Document all dependencies in the SKILL.md
4. Use relative paths from the skill directory for scripts

---
> Source: [iamzhihuix/happy-claude-skills](https://github.com/iamzhihuix/happy-claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
