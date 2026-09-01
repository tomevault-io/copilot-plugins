## grimm-os

> ﻿# AGENTS.md - Grimm App

﻿# AGENTS.md - Grimm App

This repo is the codebase. The durable project brain lives in Obsidian.

## Required Obsidian Preflight

Before changing code, architecture, prompts, schemas, memory behavior, or project direction, read:

1. `C:\Brain\10 Projects\Grimm App\Project Home.md`
2. `C:\Brain\10 Projects\Grimm App\Agent Operating Rules.md`
3. `C:\Brain\10 Projects\Grimm App\Memory System.md` when the task touches memory, persistence, AI behavior, or project context.
4. `C:\Brain\Grimm\Memory\Memory Architecture.md` when the task touches Grimm runtime memory or character memory.

Then inspect the relevant repo files and check `git status`.

## Source Of Truth Split

- Git is the source of truth for code.
- Obsidian is the source of truth for durable project memory, decisions, design notes, and AI session continuity.
- Do not store secrets in Obsidian.
- Do not put API keys in frontend code.

## After Work

- Run the smallest useful verification.
- Summarize changed files and behavior.
- Update the relevant Obsidian note when the work changes project direction, architecture, memory design, or future tasks.
- Add important decisions to `C:\Brain\40 Logs\Decision Log.md`.

## Current Important Notes

- Preserve local-first behavior unless the task explicitly changes it.
- Keep structured app memory derived from or linked back to readable notes when meaning matters.
- Start with JSON for runtime memory unless relationships or query needs justify SQLite.

---
> Source: [SaiSaiYeYint/grimm-os](https://github.com/SaiSaiYeYint/grimm-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
