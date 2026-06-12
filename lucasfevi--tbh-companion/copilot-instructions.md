## tbh-qa

> Run tbh-qa before marking app changes complete — npm run qa and dev smoke when touching main, preload, renderer, or IPC


# TBH Companion — QA before done

When you finish work under `app/`:

1. Run `cd app; npm run qa` (typecheck + tests + build + bundle path guards).
2. If you changed `main/`, `preload/`, `renderer/`, `shared/ipc.ts`, or window paths:
   - Prefer `npm run dev` and confirm the main window is **not blank** (tabs visible), or
   - Run `npm run qa:dev` when you cannot see the UI — then say visual checks were not performed.
3. **Core-only** changes (`app/src/core/` with no IPC/UI/main edits): `npm run qa` alone is enough.
4. Do not tell the user the task is done until required QA passes.

Full workflow: `.cursor/skills/tbh-qa/SKILL.md`

---
> Source: [lucasfevi/tbh-companion](https://github.com/lucasfevi/tbh-companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
