## durumu

> A browser-based practice tool for the **Light Spectrum (Colorblind) phase** from the Durumu the Forgotten encounter in World of Warcraft's Throne of Thunder raid (Mists of Pandaria).

# Durumu Light Spectrum Trainer

A browser-based practice tool for the **Light Spectrum (Colorblind) phase** from the Durumu the Forgotten encounter in World of Warcraft's Throne of Thunder raid (Mists of Pandaria).

## What This Is

An interactive HTML5 Canvas game that lets players practice the three beam-assignment roles from Durumu's Light Spectrum mechanic. Each beam has unique gameplay and scoring. The goal is to help raiders learn the mechanic before attempting it in-game.

## Tech Stack

- **Nuxt 4** (Vue 3, TypeScript, Vite) — only dependency
- **Canvas 2D** — all game rendering is native browser canvas, no graphics libraries
- **SPA mode** (`ssr: false`) — deploys to any static host (Vercel, Netlify, GitHub Pages)

## Project Structure

```
app/
├── pages/
│   ├── index.vue              # Home — beam selection menu
│   └── play.vue               # Game — orchestrates phases (instructions → countdown → play → results)
├── components/
│   ├── game/
│   │   ├── GameCanvas.vue     # Core game loop, canvas element, beam composable orchestration
│   │   ├── GameHUD.vue        # Live stats overlay (timer, beam-specific indicators)
│   │   ├── InstructionModal.vue # Per-beam instruction screens
│   │   └── ResultsScreen.vue  # Grade display, stats, feedback, retry/menu buttons
│   └── ui/
│       ├── AppHeader.vue      # Top nav bar
│       └── BeamCard.vue       # Clickable beam selection card
├── composables/
│   ├── useGameState.ts        # Global state: phase, assigned beam, score
│   ├── useGameEngine.ts       # requestAnimationFrame loop with delta-time
│   ├── useInputHandler.ts     # Mouse/touch position tracking on canvas
│   ├── useCanvasRenderer.ts   # All 2D drawing (arena, eye, beams, fogs, HUD elements, vignettes)
│   ├── useRedBeam.ts          # Red beam phase logic
│   ├── useBlueBeam.ts         # Blue beam phase logic
│   ├── useYellowBeam.ts       # Yellow beam phase logic
│   └── useScoring.ts          # Grade calculation (S/A/B/C/F) and feedback text
├── utils/
│   ├── types.ts               # TypeScript interfaces (BeamType, GamePhase, FogState, BeamCone, etc.)
│   ├── constants.ts           # Timing, colors, cone widths, scoring thresholds
│   └── geometry.ts            # Polar/cartesian math, cone collision, angle utilities
└── assets/css/main.css        # Global styles, CSS variables, dark theme
```

## Game Mechanics

### Red Beam — "Infrared Light"
- **Duration**: 50 seconds
- **Goal**: Sweep beam around arena to find and kill 3 hidden Crimson Fogs
- **Mechanic**: Hold beam on a fog to drain its health (25 HP/sec). Moving beam off a revealed fog triggers a Crimson Bloom (raid wipe). Phase ends early if all fogs are killed.
- **S-grade**: 3 kills, 0 blooms, under 30 seconds

### Blue Beam — "Blue Rays"
- **Duration**: 50 seconds
- **Goal**: Avoid revealing the single hidden Azure Fog
- **Mechanic**: Hint flashes every 8 seconds briefly show the fog's location. Revealing it triggers Icy Grasp. Repeated reveals risk killing it, causing Flash Freeze (raid wipe). Stay relatively still once you find a safe spot.
- **S-grade**: 0 reveals for the full duration

### Yellow Beam — "Bright Light"
- **Duration**: 50 seconds
- **Goal**: Stay inside an auto-rotating cone while managing Amber Fog reveals
- **Mechanic**: Beam spawns at the player's position, then auto-rotates in one direction (periodically changing speed). Time spent outside the cone accumulates damage. 2 hidden Amber Fogs (Heroic) are revealed when the cone sweeps over them — revealed fogs stay visible and must be clicked/tapped to kill before the cone passes. If a revealed fog leaves the cone alive, it triggers Burst of Amber (raid wipe = instant game over, F grade). Strategic positioning at start helps avoid fog reveals entirely.
- **S-grade**: 95%+ uptime inside the cone, 0 fogs revealed

## Commands

```bash
npm install          # Install dependencies
npm run dev          # Dev server at localhost:3000
npm run build        # Production build
npm run generate     # Static site generation (outputs to .output/public)
npm run preview      # Preview production build locally
```

## Assets

`public/images/` can optionally contain:
- `platform.png` — arena floor texture
- `boss.png` — Durumu portrait for center of arena

Both are optional. The canvas renderer falls back to programmatic drawing (gradient platform with rings, animated eye with tracking pupil) if images are missing.

## Architecture Notes

- **State flow**: `useGameState` is global (Nuxt `useState`). Phase transitions drive the entire UI: `menu → instructions → countdown → playing → results`.
- **Game loop**: `useGameEngine` runs a standard rAF loop. Each frame calls `update(deltaTime)` on the active beam composable, then `render()` via `useCanvasRenderer`.
- **Beam composables** (`useRedBeam`, `useBlueBeam`, `useYellowBeam`) share a common interface: `initialize()`, `update(dt, mouseAngle)`, `getScore()`, plus reactive state for rendering.
- **Canvas rendering** is DPR-aware and uses ResizeObserver for responsive sizing.
- **All geometry** uses polar coordinates (angle + normalized distance 0–1 relative to arena radius). Cone collision detection is in `utils/geometry.ts`.
- **Mobile support**: Touch events are handled alongside mouse events in `useInputHandler`.

## Styling

Dark theme with CSS custom properties defined in `main.css`. Beam colors: red `#ff2020`, blue `#2080ff`, yellow `#ffcc00`. Grade colors: S=gold, A=green, B=blue, C=orange, F=red. No external CSS frameworks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peopleoff)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/peopleoff)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
