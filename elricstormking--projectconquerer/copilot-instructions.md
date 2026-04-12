## projectconquerer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Charge Conquerer** is a physics-based battle simulation game designed for mobile WebGL, built with Phaser 3 and Matter.js physics engine. The game features:
- Top-down isometric 2.5D pixel art style
- Real-time physics simulation with comedic, exaggerated physics per faction
- Vampire Survivors-style level-up system during battles
- Card collection and star-rank progression meta
- Target platform: Mobile WebGL at 30 FPS lock

## Architecture & Tech Stack

- **Game Engine**: Phaser 3 (WebGL renderer)
- **Physics**: Matter.js (integrated in Phaser)
- **Target Performance**: 30 FPS lock on 2019+ mid-range mobile devices
- **Rendering**: Isometric 2:1 diamond projection with y-sort layering
- **Asset Specs**: 
  - Iso tiles: 32×16 pixels
  - Unit sprites: 32×32 (normal), 48×48 (elite/large)

## Key Systems Architecture

### Physics System
- Units use circular/sphere colliders for stability and performance
- Compound shapes only for siege units and large props
- Fixed 60 Hz physics simulation with 30/60 FPS render interpolation
- Aggressive body sleeping and damping for mobile optimization
- Maximum 160 active bodies (hard cap at 240)

### Combat System
- Damage calculation includes armor facings (front/side/rear)
- Impulse-based stagger and morale systems
- Status effects: Suppressed, Snared, Greased, Burning, Dazed
- Blast waves with 1/r² force falloff and line-of-sight checking

### Unit Management
- Deployment slots system with size-based allocation
- Class caps (Frontline/Ranged/Support/Siege/Summoner)
- Rarity tiers: Common, Rare, Epic, Legendary
- Star rank progression (★1-★5) with level caps

### Level-Up System (VS-Style)
- XP orbs collected during battle
- Time freeze on level-up with 3 skill choices
- Skills can rank up (1-5) when duplicated
- Level cap determined by star rank

## Development Commands

```bash
# Install dependencies
npm install

# Run development server with hot reload
npm run dev

# Build for production (TypeScript compile + Vite build)
npm run build

# Preview production build
npm run preview

# TypeScript type checking without emitting files
npm run typecheck

# ESLint code linting
npm run lint
```

## Performance Guidelines

- Keep draw calls under 120
- Use texture atlases for all sprites
- LOD system for particles and lights based on FPS
- Off-screen AI updates at 1/4 tick rate
- Ragdolls have 2-3 second TTL before merging/fading

## Data Architecture

The game uses designer-owned data sheets for:
- Unit stats and configurations
- Skill definitions with rank progressions
- Chest rewards and economy balancing
- Mission objectives and deployment rules

## Code Architecture

### Core Systems
- **Systems**: Located in `src/systems/` - modular game systems that can be independently updated
- **Scenes**: Located in `src/scenes/` - Phaser scenes for different game states
- **Data**: Located in `src/data/` - configuration and data definitions
- **Entities**: Located in `src/entities/` - game object classes

### Key Classes
- `BattleScene`: Main gameplay scene orchestrating all systems
- `IsometricRenderer`: Handles 2.5D isometric rendering with y-sorting
- `PhysicsManager`: Matter.js physics integration and optimization
- `UnitManager`: Unit lifecycle management and pooling
- `CombatSystem`: Damage calculation, status effects, and combat mechanics
- `DeploymentSystem`: Unit placement and formation management
- `LevelUpSystem`: Vampire Survivors-style progression system

### Scene Flow
Boot → Preload → Battle (main gameplay) + UI (overlay) + SkillSelection (pause overlay)

## Current Development Phase

**MVP (Vertical Slice)** includes:
- 6 core units
- 12 skills
- 3 maps
- 2 objective types
- 8 Tactics Rules
- Basic overlay systems
- Death ragdoll physics
- Chest reward system

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ElricStormking)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ElricStormking)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
