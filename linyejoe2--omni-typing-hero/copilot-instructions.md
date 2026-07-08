## omni-typing-hero

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start Vite dev server
npm run build     # Production build (Terser minification)
npm run deploy    # Build + deploy to Firebase Hosting
```

No test runner or linter is configured. This is a pure JavaScript project (no TypeScript, no ESLint).

## Architecture Overview

**Omni Typing HERO** is a browser-based typing RPG where players type English words to attack monsters. It uses a Canvas 2D renderer with a 60 FPS game loop, Firebase backend, and a scene-based state machine.

### Scene State Machine

The app flows through three scenes managed by `src/scenes/SceneManager.js`:

```
AuthScene → CreatorScene → BattleScene
```

Each scene extends `BaseScene` and implements lifecycle hooks: `init`, `in`, `update`, `draw`, `exit`. `SceneManager` calls these on transition and delegates `update`/`draw` each frame to the active scene.

### Game Loop

`src/main.js` owns the `GameLoop` class, which drives `requestAnimationFrame` at 60 FPS. Each tick calls:
1. `SceneManager.update()` → active scene logic
2. `CanvasManager` clears canvas → `SceneManager.draw()` → active scene rendering

### Canvas Rendering

- **gameCanvas** (800×600): main game rendering, pixel-art style (`pixelSize = 4`)
- **avatarCanvas**: hero portrait in the left panel
- **creatorCanvas**: character preview in the creator scene

All drawing is Canvas 2D API — no external renderer. Custom `util.js` helpers like `roundRect` are used throughout.

### Entity Hierarchy

```
Entity (base)
  └── BaseHero → Mage / Defender
  └── Monster → Kooni
  └── Projectile → Fireball / EnemyFireball / HandAxe
```

`heroGenerater.js` is a factory that instantiates the correct hero class by job type. Hero stats (HP, ATK, CritRate, DEF, Evasion, MagicRes) scale by level.

### Key Singletons

| Module | Location | Purpose |
|--------|----------|---------|
| `sceneManager` | `src/scenes/SceneManager.js` | Scene transitions |
| `canvasManager` | `src/ui/CanvasManager.js` | Canvas context access |
| `audioManager` | `src/services/AudioManager.js` | Web Audio API BGM/SFX |
| `dictionaryManager` | `src/services/DictionaryManager.js` | Word list (Oxford 5000) |
| `elementManager` | `src/ui/ElementManager.js` | Show/hide DOM screens |

### Typing & Combat System

`src/models/TextInput.js` handles all typing logic: WPM tracking, accuracy %, combo multipliers, frenzy mode, and DPS calculation. Each correct keystroke fires a projectile. Monsters accumulate rage and counterattack via `EnemyFireball` when thresholds are met.

Word lists come from `public/assets/cl100k_base_vocab.json` via `src/services/randomWordAPI.js`.

### Firebase Integration

`src/services/firebase.js` wraps Firebase Auth (email/password) and Firestore. Battle results and character data are persisted per user. The leaderboard (`src/ui/leaderBoard.js`) reads top-10 kills from Firestore in real time.

Firebase config is loaded from `.env.local` / `.env.production` via `VITE_FIREBASE_API_KEY`.

### Global Constants

`src/CONST.js` defines canvas dimensions, monster HP thresholds, UI layout values, and hero stat tables. Check here first when adjusting game balance or layout.

## Deployment

The `dist/` folder is served via Firebase Hosting with an SPA rewrite rule (all routes → `index.html`). Firebase project ID: `omni-typing-hero`.

---
> Source: [linyejoe2/omni-typing-hero](https://github.com/linyejoe2/omni-typing-hero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
