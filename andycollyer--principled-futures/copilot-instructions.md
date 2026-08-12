## principled-futures

> Board-level AI & ESG governance SaaS (a Salveus Labs product). Next.js 16 + React 19 + Tailwind v4.

# principled-futures-app

Board-level AI & ESG governance SaaS (a Salveus Labs product). Next.js 16 + React 19 + Tailwind v4.

## Rules
- Think before coding. Simplicity first. Surgical changes only.
- State a brief plan before multi-step work.
- `src/lib/framework.ts` is the source of truth for all assessment copy (questions, level text, band labels) — never rewrite its text.
- Answers persist via `src/lib/store.ts`, keyed by criterion id "1.1".."8.8", values 0-4. localStorage seam — to be replaced by Supabase.
- All score functions return null when unanswered — guard every render. Partial completion must never crash a screen.
- Design system: `src/components/` (ported from the design handoff) + tokens in `src/styles/tokens/`. Buttons are 8px-radius rectangles, not pills. No eyebrow labels. Tabular numerals on all scores. Transition `color`, never `background`/`all`, on nav items.
- UK English throughout. Bands: Initial / Developing / Defined / Managed / Leading.

---
> Source: [andycollyer/principled_futures](https://github.com/andycollyer/principled_futures) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
