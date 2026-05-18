## map-with-rsm-tsm-login

> 1. THE TDD MANDATE: You are strictly forbidden from writing implementation code in `app/js/` before writing a failing test in `tests/`. No exceptions.

# V7 Antigravity Global AI Rules — EVF Sales Map Rev 21.0

1. THE TDD MANDATE: You are strictly forbidden from writing implementation code in `app/js/` before writing a failing test in `tests/`. No exceptions.
2. DEPENDENCY LOGGING: Every new library requested must first be logged in `decision.md` with justification and native alternatives explored.
3. CONTEXT PINNING: Always read `architecture.md` before executing a new feature in `feature.json`.
4. STRICT PIPELINE: Intake -> Architect -> HITL Check -> Builder -> Validation. Never skip a phase.
5. NO PRODUCTION SECRETS: Refer only to `.env.example` keys. Never hallucinate or hardcode API keys.
6. THE ARCHITECTURE LOCK: If the user has marked `decision.md` as "APPROVED", you may not alter `architecture.md` without explicit permission.
7. ERROR LEARNING: Before starting any task, read `.agent/rules/error-learning.md` to check for previously learned error patterns. After resolving any error, append the error, root cause, resolution, and prevention rule to this file.
8. JSON-FIRST: Build all features against local JSON files in `data/` first. Supabase swap happens only after JSON-first features pass tests.
9. CDN-ONLY: Do NOT introduce npm/webpack/vite build tooling. All dependencies load via CDN script tags.
10. MAPLIBRE-NATIVE: Use MapLibre GL JS native clustering and source filtering. Do NOT add external clustering libraries.

---
> Source: [EVF-SPP/map-with-rsm-tsm-login](https://github.com/EVF-SPP/map-with-rsm-tsm-login) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
