## clucky-tales

> You are working on **Clucky Tales**, a browser-based cozy idle-management game about chickens.

# Clucky Tales — Cursor Rules

You are working on **Clucky Tales**, a browser-based cozy idle-management game about chickens.
Read this file before every task. It contains the project context, rules, and conventions.

## Project Context

- **Genre:** Cozy idle-management × life sim × story. Like Stardew Valley but about chickens.
- **Platform:** Browser-first (desktop Chrome/Firefox/Safari). Mobile is bonus via responsive.
- **Target audience:** Non-gamer woman who loves chickens. Everything must be cute and intuitive.
- **Core pillars:** Cozy (no stress, no punishment, no game over), personality-driven (each chicken is a character), build & discover, story matters, always a reason to return.

## Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| Engine | Phaser 3 | HTML5 game framework |
| Wrapper | Next.js (static export) | Deployed on Vercel |
| Language | TypeScript (strict) | All code in TypeScript |
| State | Zustand + localStorage | Save system |
| Localization | i18next | RU/EN bilingual |
| Art | PixelLab (MCP + manual) | HD Pixel-art |
| Deploy | Vercel | Auto-deploy on push to main |

## Art Style

- **HD Pixel-art.** Characters: 64×64. Tiles: 32×32. Buildings: 96×96–128×128.
- **Palette:** Warm, pastel, 24–32 colors. Reference: Eastward × Moonlighter × Stardew Valley.
- **Style reference file:** `public/sprites/reference_chicken.png` (when available).
- When generating art via PixelLab MCP, always use the style reference for consistency.
- PixelLab MCP docs: @https://api.pixellab.ai/mcp/docs

## Project Structure

```
clucky-tales/
├── public/
│   ├── sprites/{chickens,buildings,tiles,ui,items,effects}/
│   ├── audio/{music,sfx}/
│   ├── data/{breeds,buildings,quests,events,dialogues}.json
│   └── locales/{ru,en}/translation.json
├── src/
│   ├── app/page.tsx                    # Next.js entry point
│   ├── game/
│   │   ├── config/gameConfig.ts        # Phaser config
│   │   ├── config/constants.ts         # All balance numbers
│   │   ├── scenes/                     # Phaser scenes
│   │   ├── entities/                   # Game objects (Chicken, Egg, Building)
│   │   ├── systems/                    # Game systems (Economy, AI, Needs, etc.)
│   │   └── ui/                         # HUD, dialogs, menus
│   └── game/GameComponent.tsx          # React wrapper for Phaser
├── docs/
│   ├── GDD.md                          # Game Design Document
│   ├── SPRINT_XX.md                    # Current sprint spec
│   └── ART_GUIDE.md                    # Art pipeline guide
└── tests/
    └── balance/                        # Economy simulation tests
```

## Code Rules

### General
- All code comments in English
- All user-facing text goes through i18next (keys in JSON, never hardcoded strings)
- TypeScript strict mode, no `any` types
- Use `import`, never `require()`
- No `console.log` in production code (use a debug flag)

### Phaser-specific
- Scenes extend `Phaser.Scene`
- Entities extend `Phaser.Physics.Arcade.Sprite` or `Phaser.GameObjects.Container`
- Use object pooling for frequently created/destroyed objects (eggs, feathers, particles)
- All game balance numbers live in `config/constants.ts` — never hardcode numbers in logic
- Phaser is loaded dynamically (SSR-incompatible): use `next/dynamic` with `ssr: false`

### Systems Architecture
- Each system is a standalone class in `systems/`
- Systems communicate via Phaser events (EventEmitter), not direct references
- Game state is serializable to JSON (for save/load)
- Save format includes version number for migration

### Performance Budget
- Max 20 animated chickens on screen. Others are "inside buildings"
- Object pooling for eggs, feathers, particles
- Target: 60fps desktop. Minimum: 30fps
- Initial load: <5MB. Lazy load zones and assets
- Texture atlases (TexturePacker), not individual PNGs in production

## File Naming

- Scenes: `PascalCase` (e.g., `YardScene.ts`)
- Entities: `PascalCase` (e.g., `Chicken.ts`)
- Systems: `PascalCase` (e.g., `EconomySystem.ts`)
- Assets: `snake_case` (e.g., `chicken_silkie.png`, `grass_plain.png`)
- JSON data: `snake_case` (e.g., `breeds.json`)

## When Generating Art via PixelLab MCP

1. Always read MCP docs first: @https://api.pixellab.ai/mcp/docs
2. Use `public/sprites/reference_chicken.png` as style reference (when available)
3. Save sprites to `public/sprites/{category}/` 
4. For animated sprites: export as sprite sheet + JSON atlas (Phaser format)
5. After generating: update BootScene to load the new asset
6. After generating: update relevant JSON data file (breeds.json, buildings.json, etc.)

## When You Don't Know Something

- Game design questions → read `docs/GDD.md`
- Current sprint scope → read `docs/SPRINT_XX.md` (current sprint)
- Art style questions → read `docs/ART_GUIDE.md`
- Balance numbers → check `src/game/config/constants.ts`
- If still unclear → ask the developer, don't guess

## Don't Do This

- Don't add features not in the current sprint spec
- Don't change balance numbers without being asked
- Don't use external sprite files unless they exist in `public/`
- Don't use `localStorage` directly — use `SaveSystem`
- Don't hardcode text — use i18next keys
- Don't create separate CSS/JS files for HTML — keep everything in single files
- Don't use `THREE.CapsuleGeometry` or other APIs that don't exist in our versions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/big-bang-bean) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
