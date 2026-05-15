## comfyui-droopy-noodles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ComfyUI Droopy Noodles is a custom node extension that adds physics-based rope rendering for links in ComfyUI. It extends the built-in link rendering system with a "Physics" mode that simulates realistic rope behavior with gravity, collision detection, and smooth animation.

## Architecture

### Extension Structure

The extension consists of:
- `__init__.py` - ComfyUI custom node entry point that exposes the js directory
- `js/droopy.js` - Main extension file that patches LiteGraph's canvas rendering
- `js/RopePhysics.js` - Physics engine implementing Verlet integration for rope simulation

### Key Components

**RopePhysics.js**
- `VerletParticle` - Basic physics particle with position and velocity
- `RopeConstraint` - Distance constraint between particles
- `RopePhysics` - Individual rope simulation with:
  - Verlet integration for stable physics
  - Distance constraints to maintain rope structure
  - Rectangle collision detection for nodes
  - Penetration correction to prevent ropes passing through nodes
- `RopePhysicsManager` - Manages multiple ropes, handles creation/deletion

**droopy.js Integration**
- Registers as ComfyUI extension via `app.registerExtension`
- Adds "Physics" option to link render modes
- Overrides `LGraphCanvas` methods:
  - `renderLink` - Intercepts link rendering to apply physics
  - `drawConnections` - Manages rope lifecycle
  - `draw` - Steps physics simulation each frame

### Physics Implementation

The physics system uses:
1. **Verlet Integration** - Stable integration method for particle dynamics
2. **Distance Constraints** - Maintains rope segment lengths
3. **Collision Detection** - Prevents ropes from passing through nodes
4. **Penetration Correction** - Snaps deeply embedded segments to node corners

### Reroute Handling

For links with reroutes, the system:
- Creates separate rope segments for each link section (source→reroute, reroute→destination)
- Uses stable segment IDs that persist when nodes move
- Preserves physics state during node dragging

## Settings

The extension exposes many settings in ComfyUI:
- **Gravity**: Downward force on ropes
- **Damping**: Energy loss per frame
- **Stiffness**: Rope rigidity
- **Segments**: Number of rope particles
- **Mass**: Particle mass
- **Collision Enabled**: Toggle node collision
- **Iterations**: Constraint solver iterations
- **Debug Mode**: Visualize rope segments and collision data
- **Gradual Length Adjustment**: Smooth rope length changes
- **Length Multiplier**: Target rope length ratio
- **Length Adjust Speed**: Length change rate
- **Stretch Threshold**: Segment length multiplier to trigger correction
- **Stretch Correction Force**: Force applied to correct stretched segments

---
> Source: [guill/comfyui-droopy-noodles](https://github.com/guill/comfyui-droopy-noodles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
