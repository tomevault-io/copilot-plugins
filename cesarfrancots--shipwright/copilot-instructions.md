## shipwright

> These rules apply to all contributors and AI coding agents working in this repository.

# PM Pilot Project Rules

These rules apply to all contributors and AI coding agents working in this repository.

## Project Intent

PM Pilot is an open-source collection of Agent Skills for product managers.
The repository follows the Agent Skills open standard and aims to stay compatible with Claude Code, Codex CLI, GitHub Copilot, Cursor, and Gemini CLI.

Read `docs/PRD.md` before changing the repository architecture.

## Repository Layout

```text
pm-pilot/
├── skills/
├── docs/
├── examples/
├── install.sh
├── RULES.md
├── CLAUDE.md -> RULES.md
├── AGENTS.md -> RULES.md
├── README.md
├── CONTRIBUTING.md
└── LICENSE
```

## Skill Rules

Every `SKILL.md` must include valid YAML frontmatter with:

- `name`
- `description`
- `license`
- `metadata`

Descriptions must:

- start with an action verb
- include at least three trigger keywords
- explain the output, not only the input
- stay concise enough for reliable discovery

Write skill bodies in imperative voice.
Ask before guessing when critical context is missing.
Use `references/<file>` for longer frameworks and templates.
Keep `SKILL.md` files within the context budget.

## Content Rules

- Use ATX headers in markdown.
- Prefer one sentence per line in source where practical.
- Keep skills platform-agnostic.
- Make graceful degradation explicit when tool access is optional.
- Use Conventional Commits for commit messages.

## Installer Rules

`install.sh` must:

- auto-detect Claude Code, Codex CLI, and GitHub Copilot installations
- create missing skill directories
- use symlinks instead of copies
- be idempotent
- support `--skills`
- support `--uninstall`

## Testing Rules

Each skill must ship with at least:

- one reference file
- two test files

Tests should describe the input and expected behavior, not a brittle exact string match.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cesarfrancots) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
