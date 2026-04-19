## aetherproject

> Provides complete CSS implementation including:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Reference

- **Project**: Aether - A fully-featured incremental game with Glare Layer complete
- **Status**: Glare Layer fully implemented and functional
- **Stack**: Vue.js 3 + TypeScript + Pinia + break_infinity.js (switched from break_eternity.js)
- **Architecture**: Component-based with comprehensive state management
- **Key Rule**: Test changes thoroughly before committing - complex interconnected systems

## Project Overview

Aether is a complex incremental game with strategic depth. The Glare Layer (first of 10 layers) is fully implemented with all major systems operational including resource production, reset mechanics, cosmic filaments, constellation navigation, and advanced features like Star Memory, Automation, and Special Events.

## Project Status

**Current State**: Glare Layer fully implemented with comprehensive game systems.

**Available Files**:
- `Aether_Project_Concept.md` - High-level overview of all 10 game layers
- `Aether_Project_Glare_Layer_Concept.md` - Comprehensive design document for the Glare Layer (24,633 lines)
- `UI_Design_Ref.html` - Complete UI/CSS reference with animations and styling
- `break_eternity.js` - Library for handling large numbers (already downloaded)
- `CLI_Ref.md` - Claude CLI reference documentation

## Technology Stack (Planned)

Based on the parent directory's CLAUDE.md, this should be implemented as:
- **Framework**: Vue.js 3
- **State Management**: Pinia
- **Number Handling**: break_eternity.js (for large numbers beyond JavaScript's limits)
- **Styling**: CSS with animations and custom properties

## Project File Structure

```
Aether/
├── src/
│   ├── components/           # Vue components
│   │   ├── game/            # Game-specific components
│   │   │   ├── StarMap.vue
│   │   │   ├── ResourceDisplay.vue
│   │   │   ├── FilamentOrbit.vue
│   │   │   ├── NebulaGrid.vue
│   │   │   └── RailRoadNetwork.vue
│   │   └── ui/              # UI components
│   ├── stores/              # Pinia stores
│   │   ├── resources.ts     # Resource management
│   │   ├── filaments.ts     # Cosmic Filaments system
│   │   ├── starburst.ts     # Reset mechanics
│   │   ├── nebula.ts        # Grid system
│   │   └── gameLoop.ts      # Main game loop
│   ├── utils/               # Utilities
│   │   ├── decimal.ts       # break_eternity wrapper
│   │   ├── formatting.ts    # Number formatting
│   │   └── calculations.ts  # Game calculations
│   ├── types/               # TypeScript types
│   └── assets/              # Images, styles
├── public/                  # Static assets
├── tests/                   # Test files
└── docs/                    # Design documents
    ├── Aether_Project_Concept.md
    └── Aether_Project_Glare_Layer_Concept.md

```

## Key Design Documents

### Aether_Project_Glare_Layer_Concept.md
Contains complete specifications for:
- 15 detailed sections covering all game mechanics
- Exact formulas for calculations (e.g., Starburst multiplier: base x2, cumulative with x1.1 per Starlight)
- Reset mechanics (Starburst at 1e100 Stardust, tier unlock conditions)
- All 12 constellation effects with penalties and synergies
- UI layout specifications (Star Map system)
- Progression timeline (50+ hours to Nova Layer)

### UI_Design_Ref.html
Provides complete CSS implementation including:
- Star Map layout with central star and orbiting filaments
- Nebula grid system styling
- Rail Road constellation network visualization
- Animation keyframes for pulsation effects
- Color schemes and visual hierarchy
- Responsive design considerations

## Game Architecture

### Core Systems (Glare Layer)

1. **Resource System**
   - Stardust (basic production)
   - Starlight (prestige currency)
   - Star Rail, Nebular Essence, Stellar Energy, Cosmic Fragment

2. **Production Systems**
   - Cosmic Filaments (10 tiers with hierarchical production)
   - Milestone system (bonuses at purchase thresholds)
   - Evolution system (3 stages per filament)

3. **Reset Mechanisms**
   - Starburst (soft reset)
   - Starlight (extensive soft reset)
   - Supernova (layer transition preparation)

4. **Advanced Systems**
   - Nebula grid system with pattern bonuses
   - Star Pulsation with 5 cycle states
   - Rail Road constellation navigation
   - Star Memory preservation system
   - Upgrade tree with 4 branches

## Development Commands

```bash
# Start development server
npm run dev

# Build for production  
npm run build
npm run preview

# Type checking
npm run type-check

# Note: Linting and formatting are not configured
# npm run lint outputs "No linting configured"
# npm run format outputs "No formatting configured"
```

## Implemented Architecture

### Core Store Structure (Pinia)
- **gameState.ts** - Central game state, resources, filaments, reset logic
- **gameLoop.ts** - Main game loop and time management
- **achievements.ts** - Achievement tracking and unlocks
- **automation.ts** - Automated purchasing and progression
- **condensation.ts** - Advanced resource condensation mechanics
- **evolution.ts** - Filament evolution system (3 stages per tier)
- **memory.ts** - Star Memory preservation across resets
- **nebula.ts** - Grid-based placement system with pattern bonuses
- **pulsation.ts** - Star Pulsation 5-cycle system
- **railroad.ts** - Constellation navigation with 12 constellation types
- **starecho.ts** - Star Echo advanced system
- **upgrades.ts** - Upgrade tree with 4 branches
- **events.ts** - Special Events system
- **tooltips.ts** - Dynamic tooltip system

### Component Organization
```
components/
├── effects/        # Particle effects and animations
├── game/          # Core game mechanics UI
├── layout/        # App structure and layout
├── system/        # Error handling and settings
└── ui/           # Reusable UI components
```

### Key Files for Maintenance
- **src/utils/decimal.ts** - Wrapper for break_infinity.js number handling
- **src/utils/formatting.ts** - Number display formatting
- **src/types/game.ts** - Core type definitions
- **src/composables/** - Vue composition functions for complex logic

## Development Guidelines

### Working with Existing Systems

1. **State Management**
   - All stores are interconnected - changes in one may affect others
   - The gameState store is central - most other stores depend on it
   - Use try-catch patterns when accessing other stores (see gameState.ts examples)
   - Always test store interactions after modifications

2. **Number Handling**
   - Use `D()` function from `@/utils/decimal` for all Decimal operations
   - Import `ZERO`, `ONE` constants for common values
   - Numbers are displayed using `@/utils/formatting`
   - Never use native JavaScript numbers for game calculations

3. **Component Development**
   - Follow established naming patterns in components/
   - Use composition API with Pinia stores
   - Implement error boundaries for complex components
   - Consider mobile responsiveness (styles/mobile.css exists)

4. **Performance**
   - Game loop runs at 60 FPS via requestAnimationFrame
   - Avoid heavy computations in reactive watchers
   - Use computed properties for derived values
   - Large calculations are already optimized in stores

5. **Save System**
   - Auto-save implemented - don't duplicate
   - Save data includes all store states
   - Version compatibility handled in load() functions
   - Use base64 encoding for save strings

## Critical Game Values

### Key Formulas (from design doc)
- **Starlight Gain**: 1 Starlight when Stardust reaches 1e100
- **Star Rail Gain**: `Math.floor((starlightGained / 10) ** 0.5)` (minimum 1)
- **Filament Cost**: `baseCost * (costIncreaseFactor ** owned)`
- **Milestone Bonus**: x2 production for every 10 purchases
- **Hierarchy Synergy**: `(lowerTierQty ** 0.5) * (higherTierQty ** 0.3)`
- **Starburst Multiplier**: Base x2, cumulative (x1.1 per Starlight)

### Filament Base Values
| Tier | Base Cost | Cost Factor | Production Mult |
|------|-----------|-------------|-----------------|
| 1    | 10        | 1.8         | 1.5x            |
| 2    | 100       | 1.85        | 1.55x           |
| 3    | 1e4       | 1.9         | 1.6x            |
| 4    | 1e6       | 1.95        | 1.65x           |
| 5    | 1e9       | 2.0         | 1.7x            |
| 6    | 1e13      | 2.1         | 1.75x           |
| 7    | 1e18      | 2.2         | 1.8x            |
| 8    | 1e24      | 2.25        | 1.85x           |
| 9    | 1e31      | 2.3         | 1.9x            |
| 10   | 1e40      | 2.4         | 2.0x            |

## Common Maintenance Tasks

### Debugging Store Issues
1. Check browser console for store access errors
2. Verify store initialization order in main.ts
3. Use Vue DevTools to inspect Pinia store states
4. Look for circular dependencies between stores

### Adding New Features
1. Create appropriate TypeScript types in types/
2. Add store logic following existing patterns
3. Implement UI components with error boundaries
4. Test save/load compatibility
5. Update this CLAUDE.md if significant

### Performance Issues
1. Check game loop performance in browser DevTools
2. Look for reactive watchers causing excessive recalculation
3. Verify Decimal operations aren't using native numbers
4. Check for memory leaks in long-running sessions

### Balance Changes
1. Modify base values in gameState.ts filamentData array
2. Update formulas in computed properties and functions
3. Test progression from fresh game to late-game
4. Verify reset mechanics still work correctly

## Key Implementation Details

### Reset System Architecture
- **Starburst**: Resets filaments and stardust, keeps evolution and starlight
- **Starlight Reset**: Resets everything except Star Memory preserved items
- Star Memory system can preserve specific items across resets
- Dimensional Anchor upgrade skips every 3rd Starburst reset

### Number System
- Uses break_infinity.js instead of break_eternity.js
- All game values stored as Decimal objects
- Supports numbers up to approximately 1e1000 (not 1e10000+ as originally planned)
- Formatting handles scientific notation and suffixes

### Store Dependencies
- gameState is the root store - most systems depend on it
- Stores safely access each other using try-catch patterns
- Failed store access returns null, allowing graceful degradation
- Save/load system handles all stores automatically

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kunho817) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
