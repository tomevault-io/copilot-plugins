## canvas-effect-gal-ery

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a vanilla JavaScript visual effects gallery that renders real-time animated canvas effects. The project uses ES6 modules and runs entirely in the browser without any build tools or dependencies.

## Running the Application

Since this is a static HTML/JS/CSS application, it needs to be served via HTTP:

```bash
# Using Python 3
python -m http.server 8000

# Using Python 2
python -m SimpleHTTPServer 8000

# Using Node.js (if http-server is installed)
npx http-server -p 8000
```

Then open `http://localhost:8000` in a browser.

## Architecture

### Core Components

**main.js** - The application controller that:
- Initializes the canvas and manages the animation loop using `requestAnimationFrame`
- Handles effect switching via a dropdown selector
- Manages the sidebar UI and dynamic controls
- Calls `update()` and `draw()` on the active effect each frame
- Properly cleans up effects via `destroy()` to prevent memory leaks

**effects/** - Self-contained effect modules that follow a consistent interface:
- `constructor(canvas, ctx, controls)` - Receives canvas, context, and control container
- `update()` - Called every frame to update internal state/animations
- `draw()` - Called every frame to render to the canvas
- `resize(width, height)` - Called when window resizes
- `destroy()` - Clean up event listeners and resources
- `addControl(id, label, min, max, value, step)` - Add slider controls to sidebar

### Effect Implementations

**Plasma** (effects/plasma.js):
- Renders to an off-screen canvas at configurable resolution, then scales up for performance
- Uses mathematical sine wave combinations based on distance calculations
- Implements HSL to RGB color conversion for smooth color transitions
- Mouse/touch position influences the plasma pattern in real-time

**Starfield** (effects/starfield.js):
- Simulates 3D starfield using perspective projection (z-depth)
- Stars move based on speed and optional "warp" mode (creates streak lines)
- Mouse/touch influences direction of movement
- Stars are reinitialized when they go out of bounds or when canvas resizes

**Kaleidoscope** (effects/kaleidoscope.js):
- Composes the Plasma effect as a texture
- Uses canvas transformations (rotate, scale, clip) to create mirrored slices
- Alternates flipping slices to create symmetrical kaleidoscope patterns
- Forwards touch events to the embedded Plasma instance

### Event Handling Pattern

All effects implement consistent mouse and touch event handling:
- Events are bound in constructor using `.bind(this)` for proper context
- Touch events call `preventDefault()` to avoid page scrolling
- Multi-touch is supported by averaging touch positions
- Event listeners are removed in `destroy()` to prevent memory leaks

## Adding New Effects

To add a new effect:

1. Create a new file in `effects/` directory (e.g., `effects/myeffect.js`)
2. Export a class that implements the required interface:
   - `constructor(canvas, ctx, controls)`
   - `update()` - animation logic
   - `draw()` - rendering logic
   - `resize(width, height)` - handle canvas resize
   - `destroy()` - cleanup
3. Import the effect in `main.js`: `import { MyEffect } from './effects/myeffect.js';`
4. Add to the effects object in `main.js`: `MyEffect` (line 28)

The effect will automatically appear in the dropdown selector.

## Performance Considerations

- The Plasma effect uses an off-screen canvas rendered at lower resolution to improve performance
- The animation loop runs continuously at the browser's refresh rate
- Canvas is resized to full window dimensions, which can be expensive for complex effects
- Consider using off-screen rendering or lower resolution techniques for computationally intensive effects

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ajcates) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
