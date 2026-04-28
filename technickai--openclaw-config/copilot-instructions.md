## openclaw-config

> handles installation and updates:

# Project Context for AI Assistants

## Project Overview

Shareable configuration for OpenClaw AI assistant — templates, memory architecture, and
integration skills.

## Tech Stack

- Skills: Standalone UV scripts (Python 3.11+, inline dependencies)
- Tests: pytest via `uv run --with pytest pytest tests/ -v`
- No project-level dependencies — each skill is self-contained

## Project Structure

- `skills/` — Integration CLIs (Parallel, Limitless, Fireflies, Quo, OpenClaw)
- `templates/` — AGENTS.md, SOUL.md, USER.md templates for OpenClaw instances
- `memory/` — Example memory architecture structure
- `tests/` — Skill tests (integration tests auto-skip without API keys)

## Code Conventions

- Skills are standalone UV scripts with `# /// script` inline dependencies — no
  pyproject.toml, no shared code between skills. Each skill is fully self-contained.
- Each skill has `SKILL.md` (metadata) + executable script (same name as directory)
- Bump `VERSION` file and skill's `SKILL.md` version on changes
- Keep README.md in sync: update the skill/workflow tables when adding, removing, or
  versioning skills and workflows (counts and version badges are intentionally omitted
  from the README to avoid maintenance burden)
- Tests skip gracefully when API keys unavailable
- Keep secrets out of the repo — API keys and `.env` files stay local
- **This is a public repo — zero PII, zero fleet specifics.** This repo is a shareable
  template that anyone can clone. Never include: real names, Telegram/chat IDs, phone
  numbers, bot usernames, API keys, IP addresses, hostnames, instance IDs, security
  group IDs, or any other personally identifiable information. Use generic placeholders
  (`<ADMIN_NAME>`, `<TELEGRAM_ID>`, `<BOT_TOKEN>`, `machine-1`) in examples. Fleet-
  specific details belong in `~/openclaw-fleet/` (private) or `CLAUDE.local.md`
  (gitignored). When in doubt, use a placeholder.
- Store persistent state in markdown, not JSON — agents read and update markdown
  naturally without parsing. JSON is fine for API responses and tool output (`--json`
  flags), but state files (health reports, security posture, drift baselines) should be
  markdown

## Deployment Model

This repo is the **upstream source** for live OpenClaw instances. The `openclaw` skill
handles installation and updates:

- `templates/` → copied to instance workspace on install/update
- `skills/` → copied to instance workspace (safe to overwrite)
- `workflows/` → all upstream-owned files are synced (AGENT.md, classifier.md, platform
  guides, etc.); user-owned files (`rules.md`, `agent_notes.md`, `preferences.md`,
  `processed.md`, `logs/`) are never overwritten
- `devops/` → health check and machine setup specs, deployed via cron on fleet machines

This matters: skills and templates can be freely edited here. Workflow directories
contain a mix of upstream-owned files (algorithms, classifiers, platform guides) and
user-owned files (rules, notes, logs). The deploy logic syncs all upstream files while
preserving user state.

## Naming History

This project has been renamed twice: **Clawdbot** → **Moltbot** → **OpenClaw** (current,
after Anthropic trademark concerns). Legacy paths and config references using old names
still exist — both work, but prefer `openclaw` going forward.

## Git Workflow

Commit style: Conventional or emoji prefix, Co-Authored-By for AI commits.

Example: `Rewrite Parallel.ai skill from Bash to Python`

---
> Source: [TechNickAI/openclaw-config](https://github.com/TechNickAI/openclaw-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
