## spectrum-shift

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Spectrum Shift is an accessible web game for the "Games for Blind Gamers 5" game jam. Players explore a temple using different vision modes that simulate real visual impairments (tunnel vision, protanopia, visual snow, cataracts). The theme is "Confidently Wrong" - educating sighted users about visual impairments.

## Running the Game

No build step required. Serve with any static HTTP server:
```bash
python3 -m http.server 8080
# Then open http://localhost:8080
```

## Architecture

### Tech Stack
- Vanilla HTML/CSS/JavaScript (no framework, no build tools)
- Howler.js (CDN) for spatial audio
- Web Speech API for narration
- localStorage for save system

### Module System
All modules are IIFEs (Immediately Invoked Function Expressions) that expose themselves on `window`. Scripts must load in specific order (see `index.html` bottom).

### Event-Driven Communication
Modules communicate via `GameEvents` pub/sub system. Always use constants from `GameEvents.Events.*`:
```javascript
GameEvents.emit(GameEvents.Events.VISION_CHANGE, { mode: 'tunnel' });
GameEvents.on(GameEvents.Events.ROOM_ENTER, handleRoomEnter);
```

### Key Global Objects
- `GameEvents` - Pub/sub event emitter (scripts/core/events.js)
- `GameState` - Centralized state with path subscriptions (scripts/core/state.js)
- `VisionModes` - Vision mode definitions (scripts/data/vision-modes.js)
- `RoomData` - Room layouts and objects (scripts/data/rooms.js)
- `PuzzleData` - Puzzle configurations (scripts/data/puzzles.js)

### Module Categories

**Core** (`scripts/core/`): Event system, state management, main game controller

**Data** (`scripts/data/`): Static game content definitions

**Systems** (`scripts/systems/`):
- `vision.js` - Vision mode switching, CSS filter application, energy drain
- `navigation.js` - Room-based movement, compass, object rendering
- `audio.js` - Web Audio API synthesis (no audio files needed)
- `puzzle.js` - Puzzle logic with progress tracking
- `save.js` - localStorage persistence

**Accessibility** (`scripts/accessibility/`):
- `keyboard.js` - Full keyboard navigation
- `screen-reader.js` - ARIA live region management
- `narrator.js` - Web Speech API integration

**UI** (`scripts/ui/`): Menu, HUD, vision selector, settings, tutorial

### Vision Modes

CSS-based effects in `styles/vision-modes.css`:
- `vision-normal` - No effect
- `vision-tunnel` - Radial gradient mask (::after pseudo-element)
- `vision-protanopia` - SVG color matrix filter
- `vision-visual-snow` - Canvas noise overlay
- `vision-cataracts` - CSS blur + contrast reduction

### State Subscriptions
Use dot-notation paths for fine-grained subscriptions:
```javascript
GameState.subscribe('settings.masterVolume', (value) => { ... });
GameState.subscribe('energy', handleEnergyChange);
```

## Key Conventions

- Vision effects are CSS classes on `<body>`: `body.vision-tunnel`, `body.vision-protanopia`, etc.
- Hidden elements use `data-vision-required="mode"` attribute, revealed by CSS rules
- Energy system: drains while using non-normal vision, regenerates in normal mode
- All interactive elements must be keyboard accessible with visible focus states

---
> Source: [bloo-berries/spectrum-shift](https://github.com/bloo-berries/spectrum-shift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
