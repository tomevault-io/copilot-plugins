## was-sponsor-kit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start dev server (Vite HMR)
npm run build     # Type-check + build for production (tsc -b && vite build)
npm run lint      # ESLint
npm run preview   # Preview production build locally
```

There are no tests in this project.

## Architecture

Single-page React + TypeScript + Vite app. Almost all logic lives in one file: `src/WomenAlpineSponsorKitBuilder.tsx`.

**Tech stack:** React 19, Tailwind CSS v3, shadcn/ui components (from `src/components/ui/`), lucide-react icons. Path alias `@/` → `src/` via `vite-tsconfig-paths`.

**What it does:** A multi-step sponsor kit builder for Women Alpine (Georgian women's alpine program). The form has 11 steps (`StepKey` union type): `coreIdentity`, `problem`, `audience`, `program`, `budget`, `impact`, `visibility`, `branding`, `partnerships`, `legalRisk`, `vision`. The UI is a 3-column desktop layout: step nav | form | live preview summary. Content/defaults are in Georgian (ka).

**Data flow:**
- `DataModel` type defines the shape of all form fields across all 11 steps.
- Draft state is persisted in two places simultaneously:
  1. **localStorage** under key `was_sponsor_kit_draft_v1`
  2. **Remote API** via `/api/draft` (GET to load, PATCH per step on submit)
- On load, the app fetches from `/api/draft?draftId=<id>` and merges with the localStorage baseline, preferring the most recently updated.
- `draftId` is resolved from the `?draftId=` URL query param, defaulting to `"women-alpine-shared-draft"`.
- Final form submission POSTs to `/api/submit`.

**Backend API** (`/api/draft`, `/api/submit`) is **not** in this repository — it's an external service proxied through Vite or a separate server. No backend code lives here.

**shadcn/ui components** in `src/components/ui/` are lightly customized copies (not imported from npm). Edit them directly if UI primitives need changes.

---
> Source: [ana-kobakhidze/was-sponsor-kit](https://github.com/ana-kobakhidze/was-sponsor-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
