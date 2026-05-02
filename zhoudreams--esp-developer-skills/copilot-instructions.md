## esp-developer-skills

> This repository contains reusable ESP-IDF skills.

# AGENTS.md

This repository contains reusable ESP-IDF skills.

## Read First

- `skills/esp-idf-shared/safety-boundaries.md`
- `skills/esp-idf-shared/environment-discovery.md`
- `skills/esp-idf-shared/execution-conventions.md`
- `skills/esp-idf-shared/platform-commands.md`

## Current Repository Contents

- Shared ESP-IDF guidance lives under `skills/esp-idf-shared/`.
- Current leaf skill entry points live under `skills/esp-idf-build/`, `skills/esp-idf-flash/`, `skills/esp-idf-monitor/`, and `skills/esp-idf-docs-mcp/`.
- The initial repository rollout remains documented in `docs/superpowers/plans/2026-04-09-esp-idf-skills-v1.md`.

## Mandatory Rules

- ESP-IDF installation and its bundled Python environment are strictly read-only.
- Never install packages into the ESP-IDF Python environment.
- Never flash unless the user explicitly asks for it.
- Prefer explicit commands from `skills/esp-idf-shared/platform-commands.md`.
- Prefer a project-provided monitor script over the bundled helper when the project documents one for agent use.

---
> Source: [ZhouDreams/esp-developer-skills](https://github.com/ZhouDreams/esp-developer-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
