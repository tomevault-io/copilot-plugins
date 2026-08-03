## resume-builder

> A premium, creative resume builder web app. Local-first (no auth, no backend). Users enter their info once, then generate polished resumes in distinct designed styles, tailor them to job descriptions, run quality checks, and export pixel-accurate PDFs.

# Resume Studio — Project Guide

A premium, creative resume builder web app. Local-first (no auth, no backend). Users enter their info once, then generate polished resumes in distinct designed styles, tailor them to job descriptions, run quality checks, and export pixel-accurate PDFs.

**Roles:** Fable 5 is the architect/planner. Sonnet is the builder. This file and `docs/` are the architect's spec — follow them.

## Documentation map (read before building)

| Doc | Contents |
|---|---|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Tech stack, folder structure, routes, state management, component structure, user flow |
| [docs/DATA_MODEL.md](docs/DATA_MODEL.md) | Full TypeScript data model — copy these types verbatim into `src/types/` |
| [docs/TEMPLATES.md](docs/TEMPLATES.md) | Template engine design, the 12-template roster, creative variation logic, ATS rating system |
| [docs/INTELLIGENCE.md](docs/INTELLIGENCE.md) | Quality checker rules, bullet builder, job-description tailoring — all client-side, rule-based |
| [docs/EXPORT.md](docs/EXPORT.md) | PDF/print strategy, JSON import/export, page overflow detection |
| [docs/BUILD_PLAN.md](docs/BUILD_PLAN.md) | Phase-by-phase build checklist with exit criteria, testing checklist, future ideas |
| [docs/DECISIONS.md](docs/DECISIONS.md) | Locked architectural decisions with rationale — do not relitigate these |
| [docs/QUESTIONS.md](docs/QUESTIONS.md) | Open questions for the architect (Fable) — the escalation channel |

## Working rules for Sonnet

1. **Build in phase order** (see BUILD_PLAN.md). Do not start a phase until the previous phase's exit criteria pass. Update the checkboxes in BUILD_PLAN.md as you complete items.
2. **Don't guess on product/design decisions.** If something isn't covered by the docs: check DECISIONS.md first. If still unanswered, write the question into QUESTIONS.md (context, options, your recommended default), tell the user to relay it to Fable (or switch the session to Fable), and either stop or continue on non-blocked work. Never silently invent a product behavior.
3. **Small deviations are fine, structural deviations are not.** Renaming a component is fine. Changing the storage format, template engine design, or data model shape requires a QUESTIONS.md entry first.
4. **Verify before claiming done.** Run `npm run build` and `npm run test` at the end of every phase. For preview/print work, actually open the print route and check it.
5. **Keep the data model as the single source of truth.** All features read/write the types in `src/types/`. Never store derived data.
6. **The app UI and the resume templates are separate design worlds.** App UI = Tailwind. Resume templates = plain CSS + CSS custom properties (see TEMPLATES.md). Don't mix them.

## Quick reference

- Stack: Vite + React 18 + TypeScript (strict) + Tailwind CSS v4 + Zustand (persist) + React Router
- Storage: `localStorage`, versioned schema with migrations (`src/lib/storage/`)
- PDF export: dedicated `/print/:resumeId` route + browser print dialog (see EXPORT.md)
- Fonts: self-hosted via `@fontsource-variable/*` packages — never CDN links
- No network calls anywhere. All intelligence features are local heuristics.

---
> Source: [MercyProductions/Resume-Builder](https://github.com/MercyProductions/Resume-Builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
