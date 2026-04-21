## dnd-map-builder

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a DnD Map Builder - a web-based, interactive map builder for Dungeon Masters to create custom maps for tabletop role-playing games. The project uses Next.js 14+ with TypeScript and follows a modern React architecture.

**Live Demo**: https://zhuzh.dev
**Repository**: https://github.com/zac-computer/dnd-map-builder

## Tech Stack

- **Framework**: Next.js 14+ with App Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **State Management**: Zustand for global state (map grid, terrain, objects)
- **Canvas**: HTML5 Canvas API for map rendering
- **Data Persistence**: Local Storage API for prototype
- **Code Quality**: ESLint and Prettier

## Development Commands

Based on the PRD specifications, the following commands should be available:

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linting
npm run lint

# Run type checking
npm run type-check
```

The application runs at http://localhost:3000

## Architecture

### Directory Structure

```
src/
├── app/                 # Next.js App Router pages and layouts
│   ├── (dashboard)/     # Dashboard route group
│   ├── globals.css      # Global styles
│   └── layout.tsx       # Root layout
├── components/
│   ├── core/            # Core app components (Canvas, Toolbar)
│   └── ui/              # Reusable UI components (Button, Tooltip)
├── lib/
│   ├── hooks/           # Custom React hooks (useMapStore)
│   └── utils.ts         # Utility functions
└── stores/
    └── mapStore.ts      # Zustand store for map state
```

### Key Components

- **Canvas**: Custom React component wrapping HTML5 Canvas for map rendering
- **Toolbar**: UI for selecting tools, terrain types, and objects
- **mapStore**: Zustand store managing grid data, terrain, and object positions

### Core Features

1. **Grid-based Canvas**: Pannable/zoomable grid foundation
2. **Terrain Painting**: Brush tool for painting terrain types (grass, water, sand, stone)
3. **Object Placement**: Place predefined objects (trees, rocks, buildings)
4. **Demo Maps**: Pre-built believable DnD maps for demonstration and learning
5. **Local Persistence**: Auto-save to browser localStorage

## State Management

The application uses Zustand for state management with the following key state:

- Grid configuration and dimensions
- Terrain data for each grid cell
- Object positions and types
- Active tool and selected terrain/object
- Canvas pan/zoom state

## Canvas Implementation

The map rendering uses HTML5 Canvas API directly. Key considerations:

- Performance optimization for large grids
- Efficient redraw strategies
- Pan and zoom functionality
- Grid snapping for object placement

## Demo Maps

The application includes pre-built demo maps in `src/lib/demoMaps.ts`:

### Available Demo Maps

1. **The Riverside Tavern** (30x25 grid)
   - A cozy tavern by the river with a small village
   - Features: Stone roads, grass areas, river with sandy banks
   - Objects: Tavern building, houses, trees, rocks
   - Perfect for starting adventures

2. **Treacherous Mountain Pass** (25x20 grid)
   - A narrow mountain pass with rocky terrain
   - Features: Stone mountains, grassy valley path
   - Objects: Large boulder formations, scattered rocks, hardy mountain trees
   - Perfect for ambushes and tactical encounters

### Demo Map Implementation

- Maps are defined as structured data with terrain patterns
- Terrain uses character mapping (G=grass, W=water, S=sand, R=stone)
- Objects are positioned with coordinates and types
- Maps can be loaded via the "Load Demo" button in the toolbar

## Local Storage Schema

Map data is persisted to localStorage with structure containing:

- Grid dimensions and cell size
- Terrain data array
- Object placement array
- Last modified timestamp

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zac-computer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
