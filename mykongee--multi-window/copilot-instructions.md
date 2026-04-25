## multi-window

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Vue 3 + Vite application that creates experimental multi-window browser interactions. The application opens multiple browser windows in animated patterns (circular arrangements, sine waves) as a creative exploration of browser window APIs.

## Development Commands

- `npm run dev` - Start development server
- `npm run build` - Build for production  
- `npm run preview` - Preview production build

## Architecture

### Core Structure
- **Vue 3 SPA** using `<script setup>` composition API
- **Vite** for build tooling and development server
- **Component-based architecture** with three main interactive buttons

### Key Components
- `App.vue` - Main application layout with vertical button list
- `CircleWindowsButton.vue` - Opens 8 windows in circular pattern around randomized center point
- `NewWindowButton.vue` - Opens single window with random dimensions and position
- `SineWaveAnimationButton.vue` - Creates animated wave patterns with multiple windows

### Browser Window Management Pattern
All components follow a consistent pattern:
1. Calculate window positions and dimensions
2. Use `window.open()` with specific features (no menubar, toolbar, etc.)
3. Apply visual styling (pastel colors, rainbow hues)
4. Auto-close windows after 5-10 seconds using `setTimeout`
5. Store window references for animation control (SineWave component)

### Animation Architecture (SineWave Component)
- Maintains `animatedWindows` array with window references and metadata
- Uses `requestAnimationFrame` for smooth 60fps animations
- Implements multiple wave algorithms: static sine wave, traveling wave, true traveling wave
- Provides cleanup functions to stop animations and close windows

## Project Goals
This is a creative experiment exploring:
- Multi-window orchestration as a library concept
- Animated patterns (fireworks, waves, boids/flocking behavior)
- Visual effects with browser windows as UI primitives
- Potential for intuitive window manipulation APIs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mykongee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
