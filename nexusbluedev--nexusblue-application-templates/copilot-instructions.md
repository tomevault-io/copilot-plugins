## nexusblue-application-templates

> > Global rules loaded dynamically from Core DB via `~/.claude/CLAUDE.md` (v9.0 bootloader).

# CLAUDE.md — NexusBlue Application Templates

> Global rules loaded dynamically from Core DB via `~/.claude/CLAUDE.md` (v9.0 bootloader).
> Project-specific rules below override global rules where they conflict.

## Project Type
Infrastructure

## What This Is
Canonical source for CLAUDE.md bootloader, governance rules, hooks, and application templates used across all NexusBlue projects.

## Repo
`NexusBlueDev/nexusblue-application-templates`

## Project-Specific Rules
- This repo is the source of truth for `~/.claude/CLAUDE.md`. Changes here propagate to all projects.
- The `claude/` directory contains the bootloader and hook scripts.

## Setup Copilot
- **Slug:** `nexusblue-application-templates` | **Dashboard:** https://setup.nexusblue.ai
- Use `vault-get.sh` for ALL credentials — never grep .env.local for secrets
- Push `setup_needs` when env vars or API keys are missing
- Push `setup_blockers` when human input or decisions are needed
- Update `setup_roadmap_phases/tasks` after each completed task
- Check feedback at session start — the founder leaves notes between sessions

---
> Source: [NexusBlueDev/nexusblue-application-templates](https://github.com/NexusBlueDev/nexusblue-application-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
