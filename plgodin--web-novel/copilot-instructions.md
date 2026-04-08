## web-novel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev --turbopack` - Start development server with Turbopack for fast builds
- `npm run build` - Build production version
- `npm run start` - Start production server
- `npm run lint` - Run ESLint for code quality checks
- `npm run deploy` - Build and deploy to Firebase hosting

## Project Architecture

This is a Phoenix Wright-style visual novel engine built with Next.js, featuring investigation sequences, dialogue systems, and court records. The architecture follows a modular component-based design.

### Core Systems

**Game State Management**: Uses Zustand store (`src/lib/stores/gameStore.ts`) for centralized state including current case, scene, court record, and investigation progress.

**Case Data Structure**: Cases are JSON files (`src/cases/case-*/case.json`) containing chapters, scenes, dialogue nodes, and asset references. Each scene can contain dialogue arrays or hotspots for investigation mode.

**Scene Navigation**: 
- Dialogue scenes: Linear progression through DialogueNode arrays
- Investigation scenes: Clickable hotspots that trigger scene transitions or examinations
- Location navigation: Special scenes with hotspots representing different areas

**Asset Management**: Cases define asset mappings for backgrounds, character sprites, evidence images, and audio files. Sprites use emotion-based keys (e.g., "izzy-idle", "izzy-excited").

### Key Components

- `GameEngine.tsx` - Main game coordinator that handles scene rendering and state transitions
- `DialogueBox.tsx` - Renders character dialogue with typewriter effects
- `CharacterSprite.tsx` - Displays character images with emotion states
- `InvestigationScene.tsx` - Handles clickable hotspots for location exploration
- `HotspotOverlay.tsx` - Manages clickable areas and cursor states

### File Organization

- `/src/cases/` - Case data and assets
- `/src/components/game/` - Core game UI components
- `/src/components/investigation/` - Investigation mode components  
- `/src/components/court-record/` - Evidence and profile management
- `/src/lib/game-engine/` - Core game logic (DialogueEngine, SceneManager)
- `/src/lib/stores/` - Zustand state management
- `/src/lib/types.ts` - TypeScript interfaces for game data

### State Management Patterns

The game store handles complex state including Sets for tracking unlocked locations and examined hotspots. Actions and conditions in dialogue nodes can modify flags, add evidence/profiles, and trigger state changes.

### Asset Loading Strategy

Assets are referenced by keys in case JSON files and resolved through the assets object. This allows for lazy loading and easy case authoring without hardcoded paths in dialogue.

## Coding Guidelines

- BLUEPRINT.md is the architectural vision for the app. New app code should ALWAYS go towards what's specified in the BLUEPRINT.md. If new code would go against the blueprint, confirm with me first.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/plgodin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/plgodin)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
