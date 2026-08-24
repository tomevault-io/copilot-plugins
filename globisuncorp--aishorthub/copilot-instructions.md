## aishorthub

> AIShortHub is an operational MVP for AI short-drama monetization: content browsing, trial-to-subscription conversion, creator upload/review workflow, admin moderation, and service orders.

# AGENTS.md

## Project Mission
AIShortHub is an operational MVP for AI short-drama monetization: content browsing, trial-to-subscription conversion, creator upload/review workflow, admin moderation, and service orders.

## Architecture Rules
1. Keep Vite + React structure.
2. Favor modular pages/components/hooks; avoid large monolithic files.
3. Keep mock data centralized in `src/data/mockData.js`.
4. All new business logic should go through a hook/service layer (`src/hooks`, `src/lib`).

## Data & Integration Rules
1. Prefer Supabase-compatible schemas and naming.
2. Keep Stripe flow as checkout-session oriented; mock fallback must remain available.
3. Platform take-rate defaults to 20% and should stay configurable.

## Routing & UX Rules
1. Never break existing routes; add safely via `resolveRoute` in `src/App.jsx`.
2. Avoid blank screens: keep safe fallbacks and error boundaries.
3. Maintain dark cinematic style and responsive behavior.

## Delivery Checklist for Future Iterations
- `npm run build` must pass before finalizing.
- Update `README.md` when scope/env/schema changes.
- Document what is real integration vs mock.
- If adding new core data entities, include them in `mockData.js` + README schema section.

---
> Source: [GlobiSunCorp/AIShortHub](https://github.com/GlobiSunCorp/AIShortHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
