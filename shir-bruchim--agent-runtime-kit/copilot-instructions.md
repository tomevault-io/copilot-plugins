## agent-runtime-kit

> This repository is a configuration kit (no build steps, no tests to run). All files are Markdown (`.md`) or shell scripts (`.sh`).

# GitHub Copilot Instructions — Agent Runtime Kit

This repository is a configuration kit (no build steps, no tests to run). All files are Markdown (`.md`) or shell scripts (`.sh`).

## Coding Conventions

- Markdown files: use XML tags (`<essential_principles>`, `<intake>`, `<routing>`, `<success_criteria>`) for skill bodies — not headings
- YAML frontmatter required for skills (`name`, `description`), subagents (`name`, `description`, `tools`, `model`), commands (`description`)
- Shell scripts: `set -euo pipefail`, explicit error messages, JSON output for installer scripts
- Keep SKILL.md files lean — move detailed steps to `workflows/` subdirectories
- CORE profile must stay small and high-signal (see `PROFILES.md`)

## Branch and Commit Conventions

- Never commit to `main`. Use `feat/*`, `fix/*`, `docs/*`, `chore/*` branches
- Conventional commits: `feat(scope): description`, `fix(scope): description`
- PRs should be small and single-concern

## Adding New Content

- New skill: follow SKILL.md format with XML tags, add to PROFILES.md under correct profile
- New subagent: follow frontmatter format in `subagents/`, update README.md table
- New command: add to `commands/`, update README.md table
- New MCP server: add to `mcp/recommended-servers.json` and `mcp/SETUP.md`

## Validation

Run before committing:
```bash
python3 scripts/validate-references.py
python3 -m json.tool mcp/recommended-servers.json
bash -n scripts/check-kit-updates.sh
bash -n scripts/install-kit.sh
```

## Key Design Rules

- MCP is always opt-in — never install by default
- CORE is default — do not add to CORE without strong justification
- No duplication across rules/skills/commands
- Scripts must be deterministic + idempotent

---
> Source: [shir-bruchim/agent-runtime-kit](https://github.com/shir-bruchim/agent-runtime-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
