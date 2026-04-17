## anterix-asteroids

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Anterix Launch: Spectrum Defender is a retro arcade game combining Asteroids (rotation/thrust mechanics) with Space Invaders (wave-based enemies). The entire game is contained in a single HTML file (`index.html`) with embedded CSS and JavaScript - no build step or external dependencies required.

## Running the Game

```bash
# Open directly in browser - no build required
start index.html          # Windows
open index.html           # macOS
xdg-open index.html       # Linux
```

## Optional Sprite Generation

For DALL-E generated pixel art sprites (optional enhancement):

```bash
cd scripts
pip install -r requirements.txt
set OPENAI_API_KEY=your-key-here  # Windows
export OPENAI_API_KEY=your-key-here  # Linux/macOS
python generate_sprites.py
```

## Architecture

### Single-File Design
The game is intentionally architected as a single `index.html` file (~100KB) containing:
- **CSS** (lines 7-95): Retro CRT aesthetic with scanline effects and neon glow
- **JavaScript** (lines 106+): Complete game engine

### JavaScript Code Sections
The code is organized into clearly labeled sections:
1. **Configuration & Constants** (`CONFIG` object): All tunable game parameters
2. **Utility Functions** (`Utils` class): Math helpers, collision detection, screen wrapping
3. **Audio System** (`AudioManager` class): Web Audio API procedural 8-bit sounds
4. **Game Objects**: Player ship, enemies, projectiles, power-ups, particles
5. **Game Loop**: State management, update/render cycles, wave progression

### Key Classes
- `Utils`: Static helpers for distance, collision, screen wrapping
- `AudioManager`: Procedural sound generation (no audio files)
- `Player`: Ship with Asteroids-style rotation/thrust physics
- Enemy types: `SignalDisruptor`, `DataPacket`, `NetworkNode`, `LegacyTower`, `SpectrumJammer`
- Power-ups: `BandwidthBoost`, `SignalShield`, `SpectrumSpread`, `NetworkSurge`

### Configuration
All game balance parameters are centralized in the `CONFIG` object at the top of the script:
- Player physics (rotation speed, thrust, friction, fire rate)
- Point values per enemy type
- Power-up durations
- Color palette (neon cyan/magenta/yellow theme)
- Combo system timings

## Technical Constraints

- **No external dependencies** for core game - must work offline
- **Single HTML file** - all CSS/JS embedded
- **Target 60 FPS** on modern browsers
- **< 100KB total** (excluding optional sprites)
- **Browser support**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dbbuilder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
