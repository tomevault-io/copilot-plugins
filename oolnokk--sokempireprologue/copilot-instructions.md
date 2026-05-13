## sokempireprologue

> This is a 2D/3D game engine toolkit for "SoK Empire Prologue" - a fighting game demo with extensive editor tools. The project uses vanilla JavaScript with Three.js for 3D rendering.

# SoK Empire Prologue - AI Assistant Guidelines

## Project Overview
This is a 2D/3D game engine toolkit for "SoK Empire Prologue" - a fighting game demo with extensive editor tools. The project uses vanilla JavaScript with Three.js for 3D rendering.

**IMPORTANT: Before implementing ANY feature, search for existing code first!**

## Quick Stats
- ~204 JS/JSON files
- 25MB total size
- No build step required (vanilla JS + ES modules)
- Offline-first with CDN fallbacks

---

## Directory Structure & What Already Exists

### `/docs/` - Main Game & Editor Tools
The primary codebase. Contains the live game demo and all editor tools.

#### Key HTML Files (Editors & Tools)
- `index.html` - Main game demo
- `animation-editor.html` - Animation frame editor
- `cosmetic-editor.html` - Character cosmetics editor
- `map-editor.html` - 2D map layout editor
- `gameplay-map-editor.html` - Gameplay logic editor (spawns, patrols, etc.)
- `3Dmapbuilder.html` - 3D map builder
- `structure-editor.html` - Structure/building editor
- `map-object-editor.html` - Map object editor
- `ability-ui-test.html` - Ability UI testing tool
- `hud-arch-demo.html` - HUD architecture demo

**BEFORE creating a new editor, check if one already exists above!**

#### `/docs/renderer/` - Core 3D Rendering System
**✅ COMPLETE SYSTEMS - DO NOT RECREATE:**
- `Renderer.js` - Main WebGL renderer class
- `scene3d.js` - 3D scene management
- `visualsmapLoader.js` - 3D asset loading with caching (has cache-busting for dev mode)
- `gltfTransforms.js` - GLTF model transformation utilities
- `rendererAdapter.js` - Adapter for renderer integration
- `index.js` - Renderer module exports

#### `/docs/config/` - Game Configuration Data
**Existing Config Systems:**
- `maps/visualsmaps/` - 3D map visual configurations (GLTF models, positions, rotations)
  - `index.json` - Main visual map registry
  - `defaultdistrict3D_visualsmap.json` - Example 3D district
- `maps/gameplaymaps/` - Gameplay logic (spawn points, patrol routes, collision)
  - `defaultdistrict3d_gameplaymap.json` - Example gameplay map
- `cosmetics/` - Character appearance items (clothing, hair, accessories)
  - `cosmetics/appearance/` - Species-specific appearance options
- `abilities/` - Character abilities and skills
- `assets/` - Asset configurations (roads, sidewalks, towers, etc.)
  - `asset-index.json` - Central asset registry
- `fighter-offsets/` - Character sprite offset data
- `prefabs/` - Reusable game object templates
- `config.js` - Main configuration module

**BEFORE adding new config types, check if a similar system exists!**

#### `/docs/assets/` - Game Assets
- `3D/` - GLTF 3D models
- `fightersprites/` - Character sprite sheets
- `weapons/` - Weapon graphics
- `cosmetics/` - Cosmetic item graphics
- `audio/` - Sound effects and music
- `areas/` - Background/area graphics
- `hud/` - UI/HUD elements
- `props/` - Prop graphics
- `prefabs/` - Prefab graphics

### `/src/` - Source Code (Alternative/Shared Modules)
Some systems have versions in both `/src/` and `/docs/`. Check both!

#### `/src/renderer/` - Renderer Source
- `Renderer.js` - Core renderer (also in docs/renderer/)
- `index.js` - Module exports

#### `/src/map/` - Map Systems
**✅ EXISTING MAP SYSTEMS:**
- `MapRegistry.js` - Map registration and management
- `GeometryService.js` - Geometry/collision calculations
- `scene3d.js` - 3D scene (alternative to docs version)
- `rendererAdapter.js` - Renderer integration
- `groupLibrary.js` - Reusable object groups
- `builderConversion.js` - Map builder data conversion
- `mapBuilderConfig.js` - Map builder configuration

#### `/src/spawn/` - Spawning System
**✅ COMPLETE SPAWN SYSTEM:**
- `SpawnService.js` - Entity spawning service

#### `/src/lighting/` - Lighting Systems
**✅ EXISTING LIGHTING FEATURES:**
- `DayNightSystem.js` - Day/night cycle
- `CandleLight.js` - Dynamic candle lighting
- `TowerLightingIntegration.js` - Tower-specific lighting

#### `/src/config/` - Config Modules
- `maps/` - Map configurations
- `groups/` - Group configurations

### `/docs/vendor/` - Third-party Libraries
**✅ THREE.JS v0.160.0 ALREADY INSTALLED:**
- `three/three.min.js` - Three.js core (classic build)
- `three/three.module.js` - Three.js ES module
- `three/GLTFLoader.js` - GLTF loader (classic)
- `three/GLTFLoader.module.js` - GLTF loader (ES module)
- `three/BufferGeometryUtils.js` - Geometry utilities
- See `docs/vendor/three/README.md` for details

**DO NOT re-download or reinstall Three.js - it's already set up with offline support!**

### Root Directory Files
- Multiple `.md` files documenting past PRs and implementations
- `ancient code-monolith of truth*.html` - Legacy monolithic code (mostly deprecated)
- `package.json` - Project metadata and scripts
- `eslint.config.mjs` - ESLint configuration

### `/tools/` - Build & Utility Scripts
- `githack-url.mjs` - Generate raw.githack.com preview URLs
- `copy-map-layout.mjs` - Map layout builder tool

### `/tests/` - Test Suite
Unit tests using Node's built-in test runner.

---

## Common Patterns & Conventions

### Module System
- **ES Modules** everywhere (`import`/`export`)
- Use `.js` extensions in import paths
- No bundler - direct browser imports

### Three.js Loading Pattern
```javascript
// Always check local vendor first, then CDN fallbacks
import * as THREE from './vendor/three/three.module.js';
import { GLTFLoader } from './vendor/three/GLTFLoader.module.js';
```

### Config File Structure
Most configs are JSON with this pattern:
```json
{
  "id": "unique-identifier",
  "name": "Display Name",
  "type": "config-type",
  // ... specific properties
}
```

### 3D Asset Loading
- Use `visualsmapLoader.js` for all 3D asset loading
- Automatic caching with dev-mode cache-busting
- See `DIAGNOSIS_VISUALMAPS.md` for details

### Map Structure
Maps have TWO parts:
1. **Visual Map** (`visualsmaps/`) - 3D models, positions, graphics
2. **Gameplay Map** (`gameplaymaps/`) - Logic, spawns, collision, patrols

**DO NOT mix visual and gameplay data!**

---

## Before You Code - Checklist

### ✅ Always Do This First:
1. **Search for existing code** - Use grep/find for similar functionality
2. **Check both `/docs/` and `/src/`** - Systems may exist in either
3. **Read relevant `.md` files** - Past PRs document existing features
4. **Check config directories** - Don't recreate config systems
5. **Look for existing editors** - 10+ editor tools already exist

### ❌ Common Mistakes to Avoid:
- ❌ Reinstalling Three.js (already at v0.160.0 in vendor/)
- ❌ Creating new editors without checking existing ones
- ❌ Implementing lighting systems (day/night, candles already exist)
- ❌ Creating spawn/patrol systems (already in SpawnService + gameplay maps)
- ❌ Building new asset loaders (visualsmapLoader handles it)
- ❌ Mixing visual and gameplay map data
- ❌ Adding build tools (project is intentionally build-free)

### 🔍 Quick Search Commands
```bash
# Find files by name
find . -name "*keyword*"

# Search code content
grep -r "className\|functionName" --include="*.js"

# Check if feature exists
grep -r "featureName" docs/ src/
```

---

## Key Documentation Files

Read these FIRST when working on related features:

- `DIAGNOSIS_VISUALMAPS.md` - 3D asset loading, caching, troubleshooting
- `README.md` - Three.js setup, visual asset config, merge conflicts
- `docs/config/maps/visualsmaps/README.md` - Visual map configuration
- `docs/GAMEPLAY_MAP_EDITOR_README.md` - Gameplay map editor guide
- `docs/vendor/three/README.md` - Three.js vendor file management

---

## Development Workflow

### Testing
```bash
npm test        # Run linter + unit tests
npm run lint    # ESLint only
npm run lint:fix # Auto-fix linting issues
```

### Map Building
```bash
npm run build:map-bootstrap  # Copy map layouts
```

### Live Preview
Use `tools/githack-url.mjs` to generate preview URLs:
```bash
node tools/githack-url.mjs
node tools/githack-url.mjs --ref=branch-name
```

### Cache Management
If visual maps don't update:
- Dev mode (localhost): Just refresh (F5)
- Production: Hard refresh (Ctrl+Shift+R)
- Manual: `import('./renderer/visualsmapLoader.js').then(m => m.clearVisualsmapCache())`

---

## Code Style

### Naming Conventions
- Classes: `PascalCase`
- Functions: `camelCase`
- Constants: `UPPER_SNAKE_CASE`
- Files: Match class name or use `kebab-case`

### File Organization
- One class per file (usually)
- Group related utilities in modules
- Export from `index.js` in each directory

### Comments
- JSDoc for public APIs
- Inline comments for complex logic only
- Don't state the obvious

---

## Performance & Optimization

### Existing Optimizations
- ✅ Visual map caching system (visualsmapLoader)
- ✅ GLTF model reuse
- ✅ Offline-first Three.js loading
- ✅ Lazy loading for editors

**Before optimizing, check if the system already has caching/lazy-loading!**

---

## Security Notes

- No backend - pure client-side application
- All game data in JSON configs (easy to modify/mod)
- No authentication or user data handling

---

## When to Ask for Clarification

Ask the user if you're unsure about:
1. Which version to use (docs/ vs src/) when both exist
2. Whether to create a new editor vs. extending existing ones
3. Map structure decisions (visual vs. gameplay placement)
4. Adding new config types not listed above

---

## Final Reminder

**🚫 DO NOT RECREATE EXISTING CODE!**

This project has 200+ files. If you're about to implement a feature:
1. Search first
2. Read docs
3. Ask if unsure
4. Only then code

Following this will save tokens and prevent duplicate/conflicting code.

---
> Source: [Oolnokk/SoKEmpirePrologue](https://github.com/Oolnokk/SoKEmpirePrologue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
