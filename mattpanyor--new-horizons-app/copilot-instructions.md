## new-horizons-app

> <!-- gitnexus:start -->

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **new-horizons-app** (225 symbols, 392 relationships, 11 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## When Debugging

1. `gitnexus_query({query: "<error or symptom>"})` — find execution flows related to the issue
2. `gitnexus_context({name: "<suspect function>"})` — see all callers, callees, and process participation
3. `READ gitnexus://repo/new-horizons-app/process/{processName}` — trace the full execution flow step by step
4. For regressions: `gitnexus_detect_changes({scope: "compare", base_ref: "main"})` — see what your branch changed

## When Refactoring

- **Renaming**: MUST use `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` first. Review the preview — graph edits are safe, text_search edits need manual review. Then run with `dry_run: false`.
- **Extracting/Splitting**: MUST run `gitnexus_context({name: "target"})` to see all incoming/outgoing refs, then `gitnexus_impact({target: "target", direction: "upstream"})` to find all external callers before moving code.
- After any refactor: run `gitnexus_detect_changes({scope: "all"})` to verify only expected files changed.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Tools Quick Reference

| Tool | When to use | Command |
|------|-------------|---------|
| `query` | Find code by concept | `gitnexus_query({query: "auth validation"})` |
| `context` | 360-degree view of one symbol | `gitnexus_context({name: "validateUser"})` |
| `impact` | Blast radius before editing | `gitnexus_impact({target: "X", direction: "upstream"})` |
| `detect_changes` | Pre-commit scope check | `gitnexus_detect_changes({scope: "staged"})` |
| `rename` | Safe multi-file rename | `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` |
| `cypher` | Custom graph queries | `gitnexus_cypher({query: "MATCH ..."})` |

## Impact Risk Levels

| Depth | Meaning | Action |
|-------|---------|--------|
| d=1 | WILL BREAK — direct callers/importers | MUST update these |
| d=2 | LIKELY AFFECTED — indirect deps | Should test |
| d=3 | MAY NEED TESTING — transitive | Test if critical path |

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/new-horizons-app/context` | Codebase overview, check index freshness |
| `gitnexus://repo/new-horizons-app/clusters` | All functional areas |
| `gitnexus://repo/new-horizons-app/processes` | All execution flows |
| `gitnexus://repo/new-horizons-app/process/{name}` | Step-by-step execution trace |

## Self-Check Before Finishing

Before completing any code modification task, verify:
1. `gitnexus_impact` was run for all modified symbols
2. No HIGH/CRITICAL risk warnings were ignored
3. `gitnexus_detect_changes()` confirms changes match expected scope
4. All d=1 (WILL BREAK) dependents were updated

## CLI

- Re-index: `npx gitnexus analyze`
- Check freshness: `npx gitnexus status`
- Generate docs: `npx gitnexus wiki`

<!-- gitnexus:end -->

# Agent Instructions

## Context

Tabletop RPG galactic map app. JSON-driven content, no database.
Read CLAUDE.md first for project overview, then return here for agent-specific guidance.

## Adding Content

### New Sector

1. Create `content/sectors/[slug].json`
2. Required fields: `name`, `description`, `color` (hex), `systems` (array)
3. Optional: `nebulaColor` (hex), `vortexes` (array), `published` (boolean, default shows)
4. System pins: `{ slug, x, y }` where x(0-1200), y(0-800) canvas coords
5. Run `npm run build` after adding

### New Star System

1. Create `content/sectors/[sector-slug]/[system-slug].json`
2. Required: `name`, `star` (object with name, type, color), `bodies` (array)
3. Star: `{ name, type, color, secondaryColor? }`
4. Body: `{ id, name, type, orbitPosition(0-360), orbitDistance(0-1) }`
5. Body types: `planet | station | moon | ship | fleet | asteroid-field`
6. Planet biomes: `desert | jungle | molten | barren | irradiated | arctic | ocean | gas-giant | tropical | savanna | continental | alpine | mining | toxic | arid | ash`
7. Optional body fields: `biome`, `lore`, `kankaUrl`, `image`, `labelPosition`, `special_attribute`, `published`

### New User

Edit `data/users.json`. Fields: `username`, `password`, `group` (faction name), `role?`, `character?`

## Key Types

- `types/sector.ts` - SectorMetadata, SystemPin, VortexPin
- `types/starsystem.ts` - StarSystemMetadata, CelestialBody, Star, PlanetBiome, CelestialBodyType
- `lib/bodyColors.ts` - Biome-to-color mapping, getBodyColors()

## Server vs Client Components

- **Server** (default): pages, layouts. Can use async/await, cookies(), fs.
- **Client** (`"use client"`): GalacticMap, SectorMap, SectorMapWithPresence, WelcomeScreen, LoginPage, PresenceCard, StarSystemBackground. Use hooks, event handlers.
- **Pattern**: Server fetches data -> passes as props -> Client renders interactively.

## Auth Flow

1. POST `/api/auth/login` with `{ username, password }` -> sets `nh_user` cookie (1hr, httpOnly:false)
2. Server Components check cookie via `await cookies()`
3. Client Components read cookie from `document.cookie`
4. Presence: Client POSTs to `/api/presence` every 5s with current location

## File Naming

- Sector slugs: kebab-case, match JSON filename (e.g., `top-right`)
- System slugs: kebab-case, match JSON filename (e.g., `belliar`)
- Components: PascalCase (e.g., `GalacticMap.tsx`)
- Utilities: camelCase (e.g., `bodyColors.ts`)

---
> Source: [mattpanyor/new-horizons-app](https://github.com/mattpanyor/new-horizons-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
