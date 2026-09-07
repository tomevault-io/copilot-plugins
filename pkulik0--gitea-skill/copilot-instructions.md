## gitea-skill

> This repository contains a Gitea skill for AI coding agents. The skill lives in `skills/gitea/`.

# Agent Instructions

This repository contains a Gitea skill for AI coding agents. The skill lives in `skills/gitea/`.

## For agents working ON this repo (contributing/editing)

- The skill file is `skills/gitea/SKILL.md` -- this is the only required file
- Reference docs go in `skills/gitea/references/` -- these are loaded on demand
- Scripts go in `skills/gitea/scripts/` -- these must be executable
- Do NOT add a `README.md` inside `skills/gitea/` -- repo-level README only
- Keep SKILL.md under 5,000 words; move detail to references/
- All `tea` commands must be accurate for tea v0.15+
- No XML angle brackets in SKILL.md frontmatter
- Folder name must match the `name` field in frontmatter

## For agents USING this skill

The skill is auto-activated when a user mentions Gitea, tea CLI, or asks to manage repos, issues, PRs, releases, actions, or other Gitea resources. Run `tea --version && tea logins ls` to verify setup before executing commands.

---
> Source: [pkulik0/gitea-skill](https://github.com/pkulik0/gitea-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
