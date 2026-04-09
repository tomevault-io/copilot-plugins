## spaceshooter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Space Shooter is a mobile web game built using Pixi.js. It's a top-down, bullet hell style game where all graphics are procedurally drawn from code (no image assets).

## Technical Requirements

- **Framework**: Pixi.js for rendering
- **Language**: Modern JavaScript (latest ES features)
- **Target Platform**: Mobile web browsers
- **Development Server**: Live Server for local development
- **Architecture Principles**:
  - Loosely coupled components
  - Highly cohesive modules
  - Emphasis on visual effects

## Development Commands

### Running the Development Server
```bash
npx live-server
```
or if live-server is installed globally:
```bash
live-server
```

## Architecture Notes

### Graphics Approach
All graphics must be procedurally generated using Pixi.js drawing APIs (Graphics, shapes, etc.). No sprite sheets or image assets should be used.

### Game Structure
Since this is a bullet hell game, expect these core systems:
- Player ship with movement controls optimized for mobile touch
- Enemy spawning and wave management
- Bullet/projectile systems with collision detection
- Particle effects and visual feedback
- Score and difficulty scaling

### Code Organization
Maintain separation of concerns:
- Game entities (player, enemies, bullets) should be independent classes/modules
- Rendering logic separate from game logic
- Input handling abstracted for mobile touch events
- Effect systems decoupled from entity logic

## Mobile Considerations
- Touch controls must be responsive and intuitive
- Performance optimization crucial (mobile GPU constraints)
- Viewport scaling for different screen sizes
- Consider both portrait and landscape orientations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ajcates)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ajcates)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
