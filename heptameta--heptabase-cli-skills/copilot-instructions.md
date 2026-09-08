## heptabase-cli-skills

> Public Agent Skills package for the Heptabase CLI. Everything committed here is visible worldwide (GitHub + user installs). Treat secrets and internal details as permanent once pushed.

# heptabase-cli-skills

Public Agent Skills package for the Heptabase CLI. Everything committed here is visible worldwide (GitHub + user installs). Treat secrets and internal details as permanent once pushed.

## Layout

| Path | Purpose |
|---|---|
| [`skills/`](skills/) | Product skills shipped to users (`heptabase-cli`, …) |
| [`.claude/skills/`](.claude/skills/) | Maintainer skills (`public-repo-guard`, `release`) — edit here only |
| [`.agents/skills`](.agents/skills) | Symlink to `.claude/skills/` for Codex and other skills-compatible agents |

Do not duplicate maintainer skills under `.agents/`.

## Before you commit

1. Install the pre-commit hook once per clone (see [CONTRIBUTING.md](CONTRIBUTING.md)).
2. Follow [`.claude/skills/public-repo-guard/SKILL.md`](.claude/skills/public-repo-guard/SKILL.md) — scan staged changes and remove anything sensitive.

## Releases

Follow [`.claude/skills/release/SKILL.md`](.claude/skills/release/SKILL.md). Full process and checklists: [CONTRIBUTING.md](CONTRIBUTING.md).

---
> Source: [heptameta/heptabase-cli-skills](https://github.com/heptameta/heptabase-cli-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
