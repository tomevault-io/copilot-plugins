## gnome-kan-do

> You are working in **Gnome-Kan-Do**, a local-only Kanban board Chrome extension with a pixel-art Garden aesthetic for drag-and-drop task management.

# Gnome-Kan-Do — Agent Directives

## Identity
You are working in **Gnome-Kan-Do**, a local-only Kanban board Chrome extension with a pixel-art Garden aesthetic for drag-and-drop task management.
This project is part of the Gnomad Enterprise workspace at `/mnt/SteamDrive/ORGANIZATION/`.

## Session Start Protocol (MANDATORY)
1. Read `.agents/LOGS/SESSION_STATE.md` FIRST — this is your context from the last session.
2. If stale or missing, read the latest `.agents/LOGS/SESSIONS/HANDOFF_*.md`.
3. Run `git log -3 --oneline` to verify state matches reality.

## Stack
- **Framework**: React 18.2 + Vite 5.2 (Chrome Extension, Manifest V3)
- **Language**: JavaScript (ES Modules)
- **Key Dependencies**: @dnd-kit/core ^6.1 (drag-and-drop), lucide-react ^0.358 (icons), date-fns ^3.6 (date utilities)

## Key Commands
- Dev: `npm run dev`
- Build: `npm run build`
- Preview: `npm run preview`

## Conventions
- Extension uses `chrome.storage.local` for all persistence — no backend, no network calls.
- Vite builds multi-entry: `popup.html`, `newtab.html`, and `src/background.js` as separate entry points.
- Output assets use flat `assets/[name].[ext]` naming (no hashes) to match `manifest.json` references.
- Custom hooks in `src/hooks/` encapsulate storage logic — keep UI components free of direct `chrome.storage` calls.

## Agent Infrastructure
- `AGENTS.md` — Agent roster and responsibilities
- `.agents/CHRONICLER.md` — Session memory protocol (follow trigger rules)
- `.agents/PROJECT_SOP.md` — Full build/test/deploy procedures
- `.agents/SKILLS/` — Evolving skill files (update when you learn something)
- `.agents/DECISIONS/` — Log architectural decisions here

## Cross-Project Resources
- Swarm Core: `/mnt/SteamDrive/ORGANIZATION/02_ai_engineering/gnomad-swarm-core/`
- RAG Search: `/mnt/SteamDrive/ORGANIZATION/02_ai_engineering/workspace-rag/`
- VAULT (secrets): `/mnt/SteamDrive/ORGANIZATION/03_admin_finance/VAULT/`
- Full workspace map: `/mnt/SteamDrive/ORGANIZATION/WORKSPACE.md`
- **Skill Library**: `/mnt/SteamDrive/ORGANIZATION/SKILL_LIBRARY/` — 573 reusable skills, recipes, and patterns. See `SKILL_LIBRARY/README.md` for the catalog. Orchestrators can pull relevant skills into `.agents/SKILLS/` when beneficial.

## Chronicler Protocol
At checkpoints ("save session"), end of day ("done for the day"), or after git push:
1. Update `.agents/LOGS/SESSION_STATE.md`
2. Append to today's `.agents/LOGS/SESSIONS/HANDOFF_YYYY-MM-DD.md`
See `.agents/CHRONICLER.md` for full trigger rules.

## Self-Evolution
When you discover a better pattern or solve a hard problem, capture it:
1. Append to `.agents/SKILLS/LEARNING_LOG.md`
2. Create/update a skill file in `.agents/SKILLS/`
See `.agents/SKILLS/SELF_EVOLUTION.md` for the full protocol.

---
> Source: [davidthegnomad/Gnome-Kan-Do](https://github.com/davidthegnomad/Gnome-Kan-Do) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
