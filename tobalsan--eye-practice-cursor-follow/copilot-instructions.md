## eye-practice-cursor-follow

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a vanilla HTML/CSS/JavaScript eye coordination training application designed for amblyopia and vision therapy. The app displays a moving shape that follows waypoints across the screen with customizable parameters for therapeutic exercises.

## Development Commands

Since this is a static web application with no build system:

- **Run locally**: Open `index.html` directly in any modern web browser
- **Test**: Manual testing in browser (no automated test suite)
- **No build process required** - all files are served statically

## Architecture

### Core Components

**Single-Class Architecture**: The entire application is built around one ES6 class `EyeTrainingApp` (app.js:1) that manages all functionality:

- **Settings Management**: Persistent configuration via localStorage with real-time UI updates
- **Animation Engine**: 60 FPS animation loop using `requestAnimationFrame` with delta-time calculations
- **Physics System**: Handles movement, collision detection, and edge reflection
- **UI Controller**: Manages settings menu, keyboard shortcuts, and theme switching

### File Structure

```
cursor-follow/
├── index.html      # DOM structure and settings UI
├── styles.css      # Complete styling with light/dark themes
├── app.js          # Single EyeTrainingApp class with all logic
└── README.md       # User documentation
```

### Key Technical Patterns

**Movement System**: Two movement modes implemented in `update()` method (app.js:243):
- Angular: Instant direction changes toward targets
- Curved: Smooth turning with constant turn rate using angle interpolation

**Settings Architecture**: Each setting has a corresponding DOM element with event listeners that:
1. Update the internal `this.settings` object
2. Call `applySettings()` to update the UI/shape
3. Call `saveSettings()` to persist to localStorage

**Performance Optimizations**:
- Hardware-accelerated positioning with `translate3d()` (app.js:324)
- `will-change: transform` CSS property for shape element
- Delta-time based movement for frame-rate independence

### Shape System

Four shape types implemented via CSS classes and pseudo-elements:
- `circle`: Border-radius styling
- `square`: No border-radius 
- `cross`: CSS pseudo-elements for horizontal/vertical lines
- `cursor`: CSS triangle using border styling

### Theme System

Dual theme support (light/dark) using CSS class toggle on root `#app` element, with automatic color adaptation for all UI components including shape colors.

## Important Implementation Details

- **Bounds Calculation**: Uses `window.innerWidth/innerHeight` for full viewport coverage
- **Collision Detection**: Shape collision with screen edges triggers reflection and optional target picking
- **Keyboard Controls**: Space (pause), R (reset), T (theme toggle) - all with proper event prevention
- **Mobile Support**: Uses `env(safe-area-inset-top)` for iOS safe areas

## Settings Persistence

All user settings are automatically saved to `localStorage` under key `eyeTrainingSettings` and restored on page load, with fallback to default values if parsing fails.

---
> Source: [tobalsan/eye-practice-cursor-follow](https://github.com/tobalsan/eye-practice-cursor-follow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
