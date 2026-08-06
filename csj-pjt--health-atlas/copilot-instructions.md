## health-atlas

> - `apps/health-app` and `apps/fifth-dawn-game` must stay independently buildable.

# AGENTS

## Monorepo Rules

- `apps/health-app` and `apps/fifth-dawn-game` must stay independently buildable.
- Shared contracts go in `packages/*`.
- Do not make the game app import or query raw health records directly.
- Health-to-game data must flow through derived, game-safe linkage only.
- Treat the repository root `src/` tree as transitional legacy code after the split. Prefer editing the app-specific trees first.

## Health App

- Preserve provider sync, analytics, AI coach, social, admin, and settings behavior.
- Keep the game surface lightweight:
  - link account
  - preview derived parameters
  - show linked missions/rewards
- Do not embed the full standalone game here.

## Game App

- Keep gameplay systems isolated from the React shell where practical.
- Prefer serializable state and adapter/repository boundaries.
- Design with future desktop/EXE/Steam packaging in mind.
- Do not make progression depend on health linkage.
- `apps/fifth-dawn-game` is the standalone game product.
- `FitCraft Island` should converge into Fifth Dawn as building / settlement systems, not as a competing standalone product.
- `Pulse Frontier` stays in the health app entertainment hub for now.

## Supabase / Security

- Use migrations only.
- Keep RLS enabled on client-facing tables.
- Prefer RPCs for sensitive linkage flows.
- Avoid broad public table access unless there is a documented temporary reason.

## Entertainment

- Preserve existing arcade, strategy, and sandbox functionality.
- New major modes must be isolated modules, not hacks inside old reducers.

---
> Source: [CSJ-PJT/Health-Atlas](https://github.com/CSJ-PJT/Health-Atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
