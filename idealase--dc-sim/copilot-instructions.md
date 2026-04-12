## dc-sim

> - **Build**: `npm run build` (`tsc && vite build`)

# dc-sim

## Quick Reference
- **Build**: `npm run build` (`tsc && vite build`)
- **Dev server**: `npm run dev` → http://localhost:5173
- **Preview**: `npm run preview`
- **Deploy**: `npm run deploy` (gh-pages -d dist)
- **Typecheck**: `npx tsc --noEmit`
- **No test runner configured** — recommended: Vitest for simulation logic

## Architecture
AI Data Center: Policy vs Physics — educational simulator/game about datacenter energy constraints.

```
/src/
  /sim/                 → Pure simulation core (no React imports)
    model.ts            → Data types and interfaces
    state.ts            → State management
    dynamics.ts         → Physics/dynamics simulation
    events.ts           → Event system
    scoring.ts          → Score calculation
    tutorial.ts         → 5-stage progressive tutorial system
    archetypes.ts       → Datacenter type definitions
    regionData.ts       → Geographic region data
    rng.ts              → Seeded RNG (deterministic randomness)
  /components/          → React UI
    /Charts/            → D3.js data visualizations
    /Controls/          → User input controls
    /Dashboard/         → Main dashboard view
    /EventTicker/       → Event feed display
    /MapBoard/          → Geographic map visualization
    /RunSummary/        → Simulation run results
    /Tutorial/          → Tutorial overlay system
  /app/                 → Main App component
  /styles/              → CSS styles
```

## Key Conventions
- **Branch**: `master` (not `main`) — but CI workflow triggers on `main` push
- **Simulation/View separation**: All physics in `/sim/`, all rendering in `/components/`
- **Seeded RNG**: Deterministic randomness via `rng.ts` — do not use Math.random()
- **D3 via React refs**: D3 renders into DOM elements obtained through React refs
- **Time scale**: 1 simulation tick = 1 week
- **Tutorial**: 5 progressive stages — tutorial state managed in `sim/tutorial.ts`

## Deployment
- **URL**: GitHub Pages (via `deploy.yml` workflow or `npm run deploy`)
- **CI**: `deploy.yml` — Node 20, triggers on push to `main` (note: default branch is `master`)
- **Base path**: `/dc-sim/` (set in vite.config.ts)

## Common Pitfalls
- Default branch is `master` but CI deploys on push to `main` — merges to `main` needed for auto-deploy
- No test framework installed — add Vitest if adding tests
- D3 + React interaction: always use refs, never let D3 and React manage same DOM nodes
- Build: `tsc && vite build` (not `tsc -b`) — different from newer repos

## Existing Agent Guidance
See `.github/copilot-instructions.md` for simulation model details, D3 patterns, and architecture decisions.

## Sensitive Files
Do not read, log, or commit: any `.env` files, credentials, secrets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/idealase)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/idealase)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
