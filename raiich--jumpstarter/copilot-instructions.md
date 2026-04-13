## jumpstarter

> A template repository for Claude Code configuration. See [README.md](README.md) / [README.ja.md](README.ja.md) for details.

# Jumpstarter

A template repository for Claude Code configuration. See [README.md](README.md) / [README.ja.md](README.ja.md) for details.

## Repository Structure

Bilingual layout: Japanese (root) and English (`en/`).

```
.claude/          # Claude Code config (Japanese)
  agents/         # Custom agent definitions
  hooks/          # Hooks (conversation logging, etc.)
  rules/          # Rules
  skills/         # Skill definitions
  settings.local.json

.devcontainer/    # Dev container config

en/               # English version (mirrors root structure)
  .claude/        # Claude Code config (English)
  .devcontainer/  # Dev container config (English)

.local/           # Local working files (gitignored)
  claude/         # Conversation logs, learnings, cache
  docs/           # Design documents, etc.
```

## JA/EN Sync Rule

When modifying files under `.claude/` or `.devcontainer/`, update the corresponding file under `en/` as well (and vice versa).

- Edit a root file → update the `en/` counterpart
- Edit an `en/` file → update the root counterpart
- Adding a new file → add to both language versions
- Translate content (Japanese ↔ English). Keep config values (JSON, etc.) identical except for the language key

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raiich)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/raiich)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
