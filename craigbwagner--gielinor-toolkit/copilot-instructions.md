## gielinor-toolkit

> A desktop RS3 (RuneScape 3) companion app for PvM tracking, drop logging, wealth tracking, and goal management. See [README.md](README.md) for overview and [docs/](docs/) for detailed documentation.

# Gielinor Toolkit — Agent Instructions

## What This Is
A desktop RS3 (RuneScape 3) companion app for PvM tracking, drop logging, wealth tracking, and goal management. See [README.md](README.md) for overview and [docs/](docs/) for detailed documentation.

## Key Documentation
- [docs/architecture.md](docs/architecture.md) — project structure, crate responsibilities, data flow
- [docs/api-reference.md](docs/api-reference.md) — all RS3 API endpoints, formats, and usage notes
- [docs/data-models.md](docs/data-models.md) — Rust structs, MongoDB document shapes, wealth tracking logic
- [docs/roadmap.md](docs/roadmap.md) — milestones with detailed task breakdowns
- [docs/future-ideas.md](docs/future-ideas.md) — planned features without a milestone yet

## Quick Reference
- **Tech stack:** Rust workspace, Tauri v2, Vue 3 (Composition API + TypeScript), Pinia, MongoDB
- **RS3 display name:** "pastafartian"
- **Preferred price source:** Weird Gloop API (not the official GE API)
- **All RS3 API calls go through the Rust backend** — most lack CORS headers
- **Drop logging is manual** — RS3 doesn't expose drop logs via API
- **Boss kill tracking is hybrid** — initial count entered manually, then auto-incremented from RuneMetrics activity feed. RS3 hiscores does NOT include boss kills (OSRS only).

## Code Conventions
- No `any` type in TypeScript
- No unnecessary type casts in TypeScript
- Follow existing patterns in the codebase
- Don't add packages without asking the user first

## Git Workflow
- Don't commit unless asked. The user reviews changes first.
- Don't include co-author attribution in commits.
- Check the branch and for uncommitted changes before starting work.

## Testing
- Run existing tests after changes.
- Don't modify or remove existing tests without asking.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/craigbwagner)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/craigbwagner)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
