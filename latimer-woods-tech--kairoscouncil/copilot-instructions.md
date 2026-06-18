## kairoscouncil

> > Canonical reference for all agents, engineers, and AI tools working in this repository.

# Kairos' Council — Standing Orders

> Canonical reference for all agents, engineers, and AI tools working in this repository.
> Read `docs/ARCHITECTURE.md` — architectural constants AC-001 through AC-010 are binding, not suggestions.
> Read `docs/GAME_DESIGN.md` for the match flow, card system, and Forge mechanics.

## Mission

Kairos' Council is a mobile-first living card game where card power is calculated from
real planetary positions using Jean Meeus astronomical algorithms. Card power is not
arbitrary — it is publicly verifiable from actual transit ephemeris data and changes
constantly as planets move. The astronomy is truth; game balance is separate.

The architecture strictly separates astronomical truth (the Meeus engine, never modified
for balance) from game balance tuning (the Event Mapper layer only).

## Stack

| Layer | Technology |
|-------|-----------|
| Monorepo | pnpm workspaces + Turborepo |
| API | Cloudflare Workers + Hono — `apps/api` |
| Real-time | Durable Objects + WebSocket Hibernation API — `apps/api` |
| Database | Neon Postgres (serverless) — `packages/database` |
| Astronomical Engine | Jean Meeus algorithms — `packages/astronomical-engine` |
| Game Engine | TypeScript — `packages/game-engine` |
| Shared Types | TypeScript interfaces — `packages/shared` |
| Payments | Stripe |
| Client | Progressive Web App — `apps/web` |

## Architectural Constants (Binding — Never Override)

These constants are enforced by `pnpm validate:arch`. Violations are bugs, not trade-offs.

| ID | Constraint |
|----|-----------|
| AC-001 | The Meeus core engine is NEVER modified for game balance. Balance tuning belongs exclusively in the Event Mapper. |
| AC-002 | All game state is server-authoritative. `@kairos/web` is a display layer only. Client-side calculations that affect game outcomes are critical bugs. |
| AC-003 | Every match is locked to a `timestamp+seed`. All probabilistic elements use this seed. `Math.random()` in `game-engine` or `astronomical-engine` is a critical bug. |
| AC-004 | Transit Power formula is locked: Base Floor(40) + Transit Bonus + Aspect Bonus + Dignity Bonus. See `docs/ARCHITECTURE.md` for full formula. |
| AC-005 | `@kairos/web` MUST NOT import from `@kairos/game-engine` or `@kairos/astronomical-engine`. |
| AC-006 | `@kairos/astronomical-engine` MUST NOT import from `@kairos/game-engine`. |
| AC-009 | Transit Power recalculated every 4 hours minimum. Sub-1° transits trigger push notifications within 15 minutes. |
| AC-010 | All match history is immutable once written. No retroactive card power adjustments. |

## Hard Constraints

- No `process.env` — use Hono bindings (`c.env.VAR`) or Worker `env.VAR`
- No Node.js built-ins in Worker code — use Web APIs
- No `Math.random()` in `game-engine` or `astronomical-engine` packages (AC-003)
- `@kairos/web` must NEVER perform game or astronomical calculations (AC-002, AC-005)
- `@kairos/astronomical-engine` must NEVER import from `@kairos/game-engine` (AC-006)
- All balance changes must go through the Event Mapper layer only (AC-001)

## Package Dependency Order

```
@kairos/shared          (types, constants, errors — no internal deps)
@kairos/astronomical-engine  (depends: shared only)
@kairos/game-engine     (depends: shared — NEVER imports astronomical internals)
@kairos/database        (depends: shared)
@kairos/api             (orchestrates all packages)
@kairos/web             (depends: shared types only — NEVER performs calculations)
```

## Commands

```bash
pnpm install
pnpm test
pnpm validate:arch      # Validates package boundary constraints — must pass before any PR
pnpm typecheck          # TypeScript strict — zero errors required
```

## Session Start Checklist

1. Read `docs/ARCHITECTURE.md` — AC-001 through AC-010 are binding
2. Read `docs/GAME_DESIGN.md` — match flow, card system, Forge mechanics
3. Run `pnpm validate:arch` — confirm package boundary integrity
4. Run `pnpm test` — note baseline pass/fail
5. Check `git log --oneline -10` — understand recent changes
6. Confirm current development phase in `README.md`

## Key Docs

| Doc | Purpose |
|-----|---------|
| `docs/ARCHITECTURE.md` | Architectural constants (AC-001–AC-010), system design |
| `docs/GAME_DESIGN.md` | Match flow, card mechanics, Forge system |
| `docs/SECURITY.md` | Auth, rate limiting, anti-cheat |
| `docs/RUNBOOK.md` | Incident response, deployment |

## Commit Format

`type(scope): description`

Scopes: `engine`, `game`, `api`, `db`, `web`, `shared`
Types: `feat`, `fix`, `refactor`, `test`, `docs`, `perf`, `chore`

---
> Source: [Latimer-Woods-Tech/kairoscouncil](https://github.com/Latimer-Woods-Tech/kairoscouncil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
