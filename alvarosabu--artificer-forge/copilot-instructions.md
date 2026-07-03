## artificer-forge

> pnpm monorepo for 3D RPG games using **Nuxt 4 + TresJS** (Vue-based Three.js).

# Artificer Forge

pnpm monorepo for 3D RPG games using **Nuxt 4 + TresJS** (Vue-based Three.js).

## Quick Reference

| App/Package | Scope | Purpose |
|-------------|-------|---------|
| `apps/playground` | `@artificer-forge/playground` | Experiments sandbox |
| `apps/game` | `@artificer-forge/game` | Tabletop RPG adventure game |
| `apps/grimoire` | `@artificer-forge/grimoire` | Documentation (Docus) |
| `packages/gamelab-components` | `@artificer-forge/components` | Shared 3D components |

## Commands

```bash
# Dev servers
pnpm -F @artificer-forge/playground dev
pnpm -F @artificer-forge/grimoire dev

# Build
pnpm -F @artificer-forge/playground build
pnpm -F @artificer-forge/components build

# Lint
pnpm -F @artificer-forge/playground lint:fix
```

## Tech Stack

| Tech | Usage |
|------|-------|
| Nuxt 4 | Full-stack Vue framework |
| TresJS | Declarative Three.js (`<TresCanvas>`, `<TresMesh>`) |
| @tresjs/cientos | Pre-built 3D helpers (OrbitControls, useGLTF) |
| @nuxt/content | YAML-based entity templates |
| Pinia | Runtime game state (entities, party, inventory) |
| pnpm catalogs | Centralized versions in `pnpm-workspace.yaml` |

## Core Patterns

### Entity System
- **Templates**: YAML in `content/entities/` → queried via `queryCollection('entities')`
- **Runtime**: Pinia store (`useGameStore`) holds spawned instances
- **Flow**: `spawnFromTemplate(templateId, position)` → EntityState in store → rendered in scene

### Component Architecture
- **Smart** (experience pages): Access store, manage entity lifecycle
- **Dumb** (Character, etc.): Receive props, render model

## Key Files (Playground)

| File | Purpose |
|------|---------|
| `stores/game.ts` | Central state: entities, party, inventory, flags |
| `content.config.ts` | Nuxt Content schema for entities |
| `content/entities/**/*.yaml` | Entity templates (characters, items, interactables) |
| `composables/useEntityTemplates.ts` | Template query helpers |
| `composables/useCharacterAnimations.ts` | Animation control |
| `composables/useCharacterController.ts` | Movement facade (pointer/keyboard modes) |
| `composables/usePointerController.ts` | Click-to-move implementation |
| `components/Character.vue` | Model rendering (dumb) |

## Nuxt Content v3 Notes

```ts
// Query templates (NOT queryContent)
const template = await queryCollection('entities')
  .where('templateId', '=', 'ranger')
  .first()
```

⚠️ `id` field is reserved for file path. Use `templateId` for custom IDs.

## Dev Philosophy

**Playground → stabilize → extract to packages → document in grimoire**

---
> Source: [alvarosabu/artificer-forge](https://github.com/alvarosabu/artificer-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
