## minecraftcore-editor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Minecraft-themed sticker editor built with React, TypeScript, and Vite. Users can upload background images and overlay animated Minecraft stickers using a canvas-based editor powered by Fabric.js. The app supports features like sticker placement, scaling, rotation, and exports compositions with FFmpeg integration.

## Development Commands

**IMPORTANT**: This project requires Node.js 16 for FFmpeg compatibility.

```bash
# Switch to Node 16 (required for FFmpeg WASM)
nvm use 16

# Start development server (port 3000)
npm run dev

# Build for production
npm run build

# Build TypeScript (without bundling)
tsc -b

# Lint code
npm run lint

# Preview production build
npm run preview
```

## Architecture Overview

### Core Technologies
- **React 19** + **TypeScript** for UI components
- **Vite 4.x** as build tool (downgraded for Node 16 compatibility)
- **Fabric.js** for canvas manipulation and sticker editing
- **FFmpeg 0.11.6** (WebAssembly) with createFFmpeg API for video/image processing and exports
- **Zustand** for state management
- **Tailwind CSS** with custom design system

### Key Architecture Patterns

**State Management**: Centralized Zustand store at `src/store/useAppStore.ts` manages:
- Canvas dimensions and background images
- Sticker instances and selection state
- Fabric.js canvas references
- Responsive layout calculations

**Canvas System**: 
- Fabric.js canvas dynamically loaded to avoid SSR issues
- Responsive canvas sizing based on viewport and background image dimensions
- Background images are fitted proportionally within available space

**Component Structure**:
- `App.tsx` - Main layout with responsive header/footer/canvas/library sections
- `StickerCanvas.tsx` - Core canvas component with Fabric.js integration
- `StickerLibrary.tsx` - Grid display of available stickers
- `PhotoUploadButton.tsx` / `DownloadButton.tsx` - File I/O components

### Design System

Comprehensive design constants in `src/constants/design.ts`:
- Viewport-relative layout dimensions (8vh header, 5vh footer, 18vh sticker library)
- Dark theme color palette with semantic naming
- Responsive spacing and typography scales
- Animation and interaction patterns

### File Structure
```
src/
├── components/          # React components
├── store/              # Zustand state management
├── types/              # TypeScript interfaces
├── utils/              # Utility functions (image processing)
├── data/               # Static data (sticker assets)
├── constants/          # Design system constants
└── assets/             # Static assets
```

## Key Development Notes

### Vite Configuration
Special configuration for FFmpeg/WASM support:
- **Vite 4.x** for Node 16 compatibility
- `vite-plugin-wasm` for WebAssembly support
- Cross-origin headers configured (COOP/COEP) for SharedArrayBuffer
- `@ffmpeg/ffmpeg` included in optimizeDeps for CommonJS compatibility
- `@ffmpeg/util` and `@ffmpeg/core` excluded from optimization

### Canvas Responsiveness
Canvas sizing is viewport-aware and recalculates on window resize. The system:
1. Calculates available space after header/footer/library
2. Fits background images proportionally within constraints
3. Updates Fabric.js canvas dimensions accordingly

### Sticker System
Stickers are defined in `src/data/stickers.ts` as `StickerAsset` objects with dimensions. Runtime instances (`StickerInstance`) track position, scale, and rotation state.

### Fabric.js Integration
- Dynamic import to avoid SSR issues
- Canvas references stored in Zustand for cross-component access
- Custom event handlers for selection and manipulation

## FFmpeg Integration

### Export Functionality
- **PNG Export**: Static images from canvas using Fabric.js `toDataURL()`
- **MP4 Export**: 3-second videos with animated stickers using FFmpeg WASM
- **Automatic Detection**: Detects animated stickers (GIF/video) to determine export format
- **Background Initialization**: FFmpeg loads automatically after component renders

### FFmpeg Setup
- **Version**: 0.11.6 with createFFmpeg API (not newer 0.12+ FFmpeg class)
- **Core**: Uses single-threaded `@ffmpeg/core-st@0.11.1` for better Vite compatibility
- **Import**: Standard ES6 import works with Vite 4.x + Node 16
- **Global Instance**: Single FFmpeg instance shared across exports

## Common Issues

- **Node Version**: Must use Node.js 16 - newer versions break FFmpeg WASM
- **Vite Version**: Must use Vite 4.x - newer versions require Node 20+
- **FFmpeg Import**: Use `import { createFFmpeg }` not `new FFmpeg()` (0.11.6 vs 0.12+)
- **Canvas Scaling**: Background images must maintain aspect ratio within available viewport space
- **Fabric.js SSR**: Always dynamically import to prevent server-side rendering issues

---
> Source: [inourbubble2/minecraftcore-editor](https://github.com/inourbubble2/minecraftcore-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
