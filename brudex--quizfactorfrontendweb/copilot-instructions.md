## quizfactorfrontendweb

> QuizeFactor frontend — design skills & API alignment


**Brand:** Navy `#1D2F6F` (primary) and gold `#FFD927` (ring / accent) — see `src/style.css` (`--primary`, `--ring`, `@theme` color aliases).

When changing UI in `src/`, **read and follow** the relevant `SKILL.md` under `.agents/skills/` (same tree as `.cursor/skills`; install/update with `npx skills add https://github.com/anthropics/skills --skill frontend-design -y`):

- **frontend-design** (Anthropic `anthropics/skills`) — distinctive, production-grade UI; bold aesthetic direction; typography/color/motion/spatial guidelines in `frontend-design/SKILL.md`. Reconcile with brand tokens below (navy/gold) so the app stays on-brand while avoiding generic “AI slop” layouts.
- **frontend-design-system** — design tokens, spacing (8px grid), accessible contrast, motion restraint.
- **shadcn** — component patterns (card, button, badge), radius and border semantics; this app uses Tailwind + CSS variables in the same spirit.
- **web-design-guidelines** — responsive layout, focus states, readable line length.

The REST surface is defined by `QuizeFactor API.postman_collection.json` at the repo root; keep `src/api/quizFactorApi.js` in sync with new endpoints.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brudex) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
