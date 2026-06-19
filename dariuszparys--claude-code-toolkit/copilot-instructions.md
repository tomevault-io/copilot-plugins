## claude-code-toolkit

> Claude Code plugin marketplace with custom commands, skills, and hooks.

# Project: claude-code-toolkit

Claude Code plugin marketplace with custom commands, skills, and hooks.

## Stack
- Claude Code plugins (commands, skills, hooks)
- Markdown with YAML frontmatter
- JSON for plugin manifests (plugin.json, marketplace.json)

## Structure
- `plugins/<name>/` — Individual plugins with `.claude-plugin/plugin.json`
- `plugins/<name>/hooks/` — Shell scripts for hook commands
- `.claude-plugin/marketplace.json` — Plugin registry

## Commands
- `tree -L 3 plugins/` — View plugin structure

## Rules
- Commands: `plugins/<name>/commands/<cmd>.md` with YAML frontmatter
  - Frontmatter supports: `description`, `argument-hint`, `allowed-tools`
  - Use subdirectories only for logical grouping (e.g., `commands/deploy/rollout.md`)
- Skills: `plugins/<name>/skills/SKILL.md`
- Hooks: Defined in `plugin.json` under `hooks` key
  - Use regex matchers (e.g., `Edit|Write`)
  - Scripts use `${CLAUDE_PLUGIN_ROOT}` for plugin-relative paths
  - Use `once: true` for hooks that should run once per session
- Keep commands focused on a single task
- Commits follow Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`)
- Version bumps require CHANGELOG.md update (Keep a Changelog format)

## Domain
- **Plugin**: Package containing commands, skills, and/or hooks
- **Command**: Markdown file invoked via `/plugin:command`
- **Skill**: Auto-invoked behavior in `SKILL.md`
- **Hook**: Shell command triggered by tool events (PostToolUse, etc.)
- **Marketplace**: Repo with `.claude-plugin/marketplace.json` listing plugins

---
> Source: [dariuszparys/claude-code-toolkit](https://github.com/dariuszparys/claude-code-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
