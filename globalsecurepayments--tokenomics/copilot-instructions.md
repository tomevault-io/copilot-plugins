## tokenomics

> > **Last Updated:** 2026-03-25

# Tokenomics Lab — AI Context

> **Last Updated:** 2026-03-25
> **Current Phase:** Sprint 1 — Foundation + CLD Editor

---

## Project Overview

Blockchain-agnostic tokenomics predictive modeling system. Implements the full modeling lifecycle: causal loop diagrams → stock-and-flow models → bonding curves & emission schedules → Monte Carlo / agent-based simulation → real-time health monitoring → 3D visualization.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Monorepo | pnpm workspaces (3 packages) |
| Frontend | SvelteKit 2 + Svelte 5 runes + Tailwind 4 + bits-ui |
| 3D | @threlte/core + @threlte/extras (Three.js) |
| Charts | Chart.js 4, D3.js |
| Graph Editor | @xyflow/svelte |
| Backend | Hono 4.7+ on Cloudflare Workers |
| Storage | Durable Objects (SQLite), R2, KV |
| Schemas | Zod (shared package) |

---

## Project Structure

```
packages/
  shared/    # @tokenomics/shared — Zod schemas, types, pure math functions
  workers/   # @tokenomics/workers — Hono API + Durable Objects (ProjectState, SimulationEngine)
  web/       # @tokenomics/web — SvelteKit frontend
```

---

## Rules

1. **Svelte 5 runes only** — `$state`, `$derived`, `$effect`. No legacy stores.
2. **Shared math is pure** — No side effects, no I/O. Functions take numbers, return numbers.
3. **Blockchain-agnostic** — No chain-specific code. Models are pure math.
4. **DO SQLite for persistence** — ProjectState stores model data, SimulationEngine stores sim results.
5. **Simulation runs server-side** — DOs use alarm-driven batching for Monte Carlo/ABM.
6. **Three.js via threlte** — All 3D scenes use `<Canvas>` from @threlte/core, never raw Three.js in Svelte components.
7. **WebSocket hibernation** — Real-time updates use `ctx.acceptWebSocket()` pattern (not polling).

---

## Commands

```bash
# Development
pnpm dev           # Frontend dev server
pnpm dev:api       # Workers dev server (wrangler)

# Build & Check
pnpm build         # Build all packages
pnpm type-check    # TypeScript check all packages
```

---

## Key Files

- `packages/shared/src/schemas/` — All Zod schemas (project, CLD, stock-flow, curves, etc.)
- `packages/shared/src/math/` — Pure math functions (bonding curves, emission, vesting, health indicators, PRNG)
- `packages/workers/src/durable-objects/project-state.ts` — Per-project model persistence
- `packages/workers/src/durable-objects/simulation-engine.ts` — Simulation runner (alarm-driven)
- `packages/workers/wrangler.jsonc` — Cloudflare Workers config
- `packages/web/src/routes/project/[id]/` — Project editor pages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/globalsecurepayments)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/globalsecurepayments)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
