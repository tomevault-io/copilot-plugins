## infinite-grid-navigation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A performant infinite draggable grid built with Next.js 15, React 19, and Motion (Framer Motion). The grid dynamically loads tiles based on viewport visibility, creating an endless scrolling experience in all directions.

## Commands

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

## Architecture

### Core Components

**InfiniteGrid** (`components/infinite-grid/index.tsx`)
- Main container component (~300 lines)
- Client component using Motion's drag controls and motion values
- Manages item lifecycle: tracks which tiles to render based on drag position
- Uses ResizeObserver for responsive reflow
- Item states: 'initializing' → 'idle' → 'dragging' (prevents click events during drag)

**GridItem** (inside same file)
- Memoized child component for individual tiles
- Animates opacity on mount, positioned via Motion's x/y transforms

### Grid Mathematics (`components/infinite-grid/utils.ts`)

- **Fixed tile size:** 300px (ITEM_SIZE constant)
- **Base matrix generation:** Creates 2D matrix with alternating pattern (item + null) for visual spacing
- **Infinite wrapping:** Uses `wrap(min, max, value)` function to enable seamless infinite scroll
- **Adjacency matrix:** 8-point clockwise matrix for surrounding tile detection
- **Viewport collision:** Only renders tiles visible in viewport plus adjacent tiles

### State Management Pattern

Uses React hooks with Motion value events:
- `useState` for item tracking
- `useMotionValueEvent` on X/Y coordinates to trigger item updates on drag
- `useLatest` hook (`lib/utils/common.ts`) to avoid stale closures in event handlers

### Path Alias

`~/*` maps to project root (configured in tsconfig.json)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/malikkotb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/malikkotb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
