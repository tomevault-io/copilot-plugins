## gayishgame

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A digital board game implementation based on the "Gayish" podcast about gay stereotypes. The game is a card game with board elements, combining card mechanics with board positioning similar to games like Ticket to Ride.

**Tech Stack:** Godot Engine (GDScript/C#)
**Game Type:** Card game with board elements
**Target Platform:** Cross-platform (desktop initially, mobile potential)

## Development Setup

**Prerequisites:**
- Godot Engine 4.x (download from godotengine.org)
- Git for version control

**Getting Started:**
1. Open the project in Godot Editor: `godot -e project.godot` (or use GUI)
2. Run the game from editor: Press F5 or click the Play button
3. Run a specific scene: Press F6

## Project Structure

```
/
├── scenes/          # Godot scene files (.tscn)
│   ├── main/       # Main game scenes
│   ├── cards/      # Card-related scenes
│   └── board/      # Board-related scenes
├── scripts/         # GDScript files (.gd)
├── assets/          # Game assets
│   ├── art/        # Sprites, textures, UI elements
│   ├── audio/      # Sound effects and music
│   └── fonts/      # Font files
├── data/            # Game data (JSON/CSV)
│   ├── cards.json  # Card definitions
│   └── prompts.json # Game prompts/questions
└── project.godot    # Godot project file
```

## Architecture

**Game Structure:**
- **Card System:** Cards with various types (challenge, movement, event) drawn from deck
- **Board System:** Grid or path-based board where players move pieces
- **Turn Management:** Turn-based gameplay with phase system
- **Multiplayer:** Local multiplayer (2-6 players recommended)

**Core Scripts Organization:**
- `GameManager.gd` - Central game state and flow control
- `CardManager.gd` - Card deck, drawing, and card effects
- `BoardManager.gd` - Board state, piece movement, and positioning
- `Player.gd` - Player data, hand management, and actions
- `UIManager.gd` - UI updates and player interactions

## Key Commands

**Development:**
- Run game: F5 in Godot Editor
- Run current scene: F6 in Godot Editor
- Export project: Project > Export (configure export presets first)

**Testing:**
- Use Godot's built-in debugger (Debug menu)
- Print debugging: `print()` or `print_debug()`
- Remote debugging available for exported builds

## Content Guidelines

The game content is based on the Gayish podcast themes. When adding cards, prompts, or game content:
- Keep the tone playful and educational about gay culture and stereotypes
- Ensure content is inclusive and respectful while being entertaining
- Reference podcast episodes or recurring themes where appropriate
- Balance challenge/trivia cards with movement/strategy cards

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/perplexingcabinet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
