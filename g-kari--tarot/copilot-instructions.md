## tarot

> - **Repo**: g-kari/tarot

# Tarot — CLAUDE.md

## Project
- **Repo**: g-kari/tarot
- **Framework**: vinext (Vite + Next.js App Router → Cloudflare Workers)
- **Package manager**: pnpm (always use pnpm, never npm/yarn)

## Commands
```bash
pnpm dev       # dev server at localhost:3000
pnpm build     # production build
pnpm deploy    # build + deploy to Cloudflare Workers (requires wrangler login)
```

## Project Structure
```
src/app/
├── data/          # cards.ts (78 cards), spreads.ts
├── store/         # useTarotStore.ts (zustand)
├── animations/    # variants.ts (framer-motion)
└── components/
    ├── TarotApp.tsx           # DndContext root ('use client')
    ├── background/
    ├── card/
    │   ├── CardShell.tsx      # 3D flip (rotateY)
    │   ├── CardBack.tsx       # SVG card back
    │   ├── CardFront.tsx      # SVG card front
    │   └── svg/
    │       ├── MajorArcanaArt.tsx   # 22 abstract SVG scenes (switch)
    │       ├── MinorArcanaArt.tsx   # pip layout + court figures
    │       ├── SuitSymbol.tsx       # ♦♠♥♣ SVG symbols
    │       └── BorderFrame.tsx
    ├── deck/      # DeckPile, DeckControls
    ├── spread/    # SpreadLayout, SlotDropZone, SpreadSelector
    └── table/     # ReadingTable (main canvas)
```

## Key Decisions
- All interactive components must have `"use client"` at the top
- `wrangler.toml` (not .jsonc/.yml — wrangler doesn't support .yml)
- Card size: 120×210px
- vinext auto-configures Vite — no vite.config.ts needed
- DnD: @dnd-kit/core with DragOverlay portal pattern
- Animations: framer-motion v12 — `layoutId` for deck→slot travel

## Deploy Architecture (vinext 0.0.18)
- `pnpm deploy` = `vinext deploy` which uses `@cloudflare/vite-plugin` internally
- `wrangler.toml` must set `main = "worker/index.ts"` (TypeScript source entry, NOT `dist/server/index.js`)
- vinext deploy auto-generates `worker/index.ts` + `vite.config.ts` if missing, then runs a 5-step Vite build
- Build output: `dist/server/` (worker bundle) + `dist/client/` (static assets)
- `@cloudflare/vite-plugin` also generates `.wrangler/deploy/config.json` (redirect file) — this is gitignored
- `@emotion/is-prop-valid` must be in `dependencies` (framer-motion optional peer dep, required at build time)
- If CI deploy fails with "compatibility date" error: purge Cloudflare Pages build cache (stale `.wrangler/deploy/config.json`)

## Design System
| Token | Value |
|---|---|
| card major border | `#c8a84b` |
| card wands border | `#e67e22` |
| card cups border | `#3498db` |
| card swords border | `#95a5a6` |
| card pentacles border | `#27ae60` |
| accent purple | `#6366f1` |
| accent gold | `#a89060` |

## Do Not
- Do not add `type="module"` to individual script tags manually
- Do not create `vite.config.ts` (vinext handles it)
- Do not use `npm` or `yarn`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/g-kari)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/g-kari)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
