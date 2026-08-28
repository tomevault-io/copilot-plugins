## al-hadi-school

> **Stack:** React + Vite · React Router · Tailwind CSS v4 · shadcn/ui (JS mode) · Supabase (Auth · DB · RLS · Storage) · Vercel. **JavaScript only — no TypeScript.**

# School Management System — Agent Rules

**Stack:** React + Vite · React Router · Tailwind CSS v4 · shadcn/ui (JS mode) · Supabase (Auth · DB · RLS · Storage) · Vercel. **JavaScript only — no TypeScript.**

> A **Vite SPA**, not Next.js: no server, no `app/`, no Server Actions. Tailwind v4 and React Router differ from older versions in your training data — verify against `context/library-docs.md` and current docs before writing code. Security is enforced by Supabase **RLS**, never the UI.

## Read Before Anything Else

In this exact order, before any implementation:

1. context/project-overview.md
2. context/architecture.md
3. context/ui-tokens.md
4. context/ui-rules.md
5. context/ui-registry.md
6. context/code-standards.md
7. context/library-docs.md
8. context/build-plan.md
9. context/progress-tracker.md

## Rules That Never Change
- never go inside docs folder and dont touch nodemodules
- JavaScript only — never TypeScript or `.ts`/`.tsx`
- No hardcoded hex or raw Tailwind color classes — use tokens (primary `#1C74BD`)
- All DB access goes through `services/` + query hooks — components never call Supabase; the service-role key never reaches the browser
- Security is RLS, not the UI: only the assigned Class Teacher enters marks, only Admin approves/publishes, only published results are public
- Update `progress-tracker.md` and `ui-registry.md` after every feature
- Before any third-party library — load its installed skill first, then read `context/library-docs.md`
- If the same problem persists after one corrective prompt — stop and run /recover

## Available Skills

- `/architect` — before any complex feature. Think before building.
- `/imprint` — after any new UI component. Capture patterns.
- `/review` — before demo or when something feels off.
- `/recover` — when something breaks after one failed correction.
- `/remember save` / `/remember restore` — for features spanning multiple sessions.

---
> Source: [SaimRaza885/Al_Hadi_School](https://github.com/SaimRaza885/Al_Hadi_School) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
