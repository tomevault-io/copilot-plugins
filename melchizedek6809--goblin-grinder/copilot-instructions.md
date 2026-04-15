## goblin-grinder

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Goblin Grinder is a WebGL2-based 3D game built with TypeScript and Vite. The project features a custom renderer with shadow mapping, GLB model loading, and spring physics for camera movement. The goal is to experiment with various render passes and shaders to achieve a pixel art look with quality lighting.

## Build and Development Commands

- **Start development server**: `npm run dev`
- **Build for production**: `npm run build` (runs TypeScript compiler followed by Vite build)
- **Preview production build**: `npm run preview`
- **Type check**: `npm run typecheck` (TypeScript type checking without emit)
- **Lint**: `npm run lint` (Biome linter with auto-fix)
- **Format**: `npm run format` (Biome code formatter)

## Architecture

### Core Structure

**Entry Point**: `src/main.ts`
- `Game` class manages the entire application lifecycle
- Handles WebGL2 context initialization
- Manages the main render loop via `requestAnimationFrame`
- Coordinates player input, camera updates, and scene rendering

### Rendering Pipeline

The renderer uses a multi-pass approach:

1. **Shadow Pass** (Light.ts) - For each light source:
   - Render scene to shadow map framebuffer (2048x2048 depth texture)
   - Uses depth-only shaders (depth.vert/depth.frag)
   - Orthographic projection from light's perspective

2. **Main Pass** (Camera.ts) - From camera perspective:
   - Bind shadow maps from all lights
   - Render scene with basic.vert/basic.frag shaders
   - Apply lighting, shadows, and textures

### Key Classes

**Mesh** (Mesh.ts)
- Manages VAO, vertex buffers, and index buffers
- Vertex format: position(3) + normal(3) + uv(2) + color(3) = 11 floats
- Optional texture support with `loadTexture()` method
- Factory methods: `createCube()`, `createPlane()`, `fromMeshData()`

**Entity** (Entity.ts)
- Game object with transform: position (vec3), rotation (quaternion), scale (vec3)
- Generates model matrices for rendering
- References a Mesh for geometry

**Player** (Player.ts)
- Controls player character (can have multiple entities/meshes)
- Handles movement with `move(x, z, deltaTime)`
- Automatically rotates character to face movement direction
- Move speed: 3.0 units/second

**Camera** (Camera.ts)
- Perspective projection with configurable FOV
- Spring physics follow system with configurable stiffness and damping
- `setFollowTarget()` - Sets goal position based on player
- `updateFollow()` - Smooth camera movement with overshoot
- Manages render passes and light/shadow uniforms

**Light** (Light.ts)
- Directional light with shadow map support
- Configurable orthographic shadow projection
- `renderShadowMap()` - Renders depth pass for shadows
- Supports up to 4 lights simultaneously

**Shader** (Shader.ts)
- Compiles and links vertex/fragment shaders
- Manages uniform locations with caching
- Helper methods for setting uniforms (matrices, vectors, ints, floats)
- Array uniform support for lights and shadow maps

**GLBLoader** (GLBLoader.ts)
- Custom GLB/glTF 2.0 parser (no external dependencies)
- Returns format-agnostic `MeshData` interface
- Extracts: positions, normals, UVs, colors, indices
- Handles multiple primitives/meshes per file

### Shader System

**Vertex Shader** (basic.vert)
- Inputs: a_position, a_normal, a_uv, a_color
- Outputs: v_worldPosition, v_normal, v_uv, v_color, v_lightSpacePositions[4]
- Transforms vertices to light space for shadow mapping

**Fragment Shader** (basic.frag)
- Samples texture if available, otherwise uses vertex color
- Calculates shadows for each light using shadow maps
- Applies diffuse lighting with configurable ambient
- Unrolled light loop (GLSL limitation: sampler arrays need constant indices)

**Depth Shaders** (depth.vert/depth.frag)
- Minimal shaders for shadow map generation
- Only outputs depth information

### Input System

**Keyboard Controls** (main.ts)
- WASD: Move player (relative to camera angle)
- Q/E: Rotate camera 90° with spring physics
- Input handling uses Set for simultaneous key presses
- Movement normalized for consistent diagonal speed

### Camera System

**Spring Physics**
- Separate springs for camera position and target
- Configurable stiffness (20.0) and damping (0.7)
- Creates smooth follow with slight overshoot
- Camera offset calculated based on rotation angle (5 units distance, 8 units height)

**Rotation**
- Rotates in 90° increments around player
- Uses spring physics for smooth rotation with overshoot
- Stiffness: 40.0, Damping: 0.6

### Asset Loading

**Models**: Placed in `src/assets/models/`
- GLB format preferred (single file)
- Texture loading via `Mesh.loadTexture(url)`
- Multiple meshes per model supported (e.g., character body parts)

**Shaders**: Placed in `src/shaders/`
- Imported with `?raw` suffix for syntax highlighting
- Separate .vert and .frag files

## TypeScript Configuration

- **Target**: ES2022
- **Module system**: ESNext with bundler resolution
- **Strict mode**: Enabled with additional linting rules
- **Experimental decorators**: Enabled (for Lit compatibility, currently unused)
- **No emit**: TypeScript only for type checking; Vite handles transpilation

## Dependencies

- **gl-matrix**: Vector and matrix math operations
- **lit**: UI framework (included but not currently used)
- **@biomejs/biome**: Fast linting and formatting

## Important Notes

- Renderer is completely custom - no game engine used
- All geometry uses interleaved vertex format for efficiency
- Shadow maps use texture slot 0-3, material textures use slot 4+
- Camera follows player position, not hardcoded to specific coordinates
- Player rotation automatically faces movement direction
- Spring physics used for camera follow and rotation for dynamic feel
- Never start a dev server yourself, I have one always running in another terminal

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Melchizedek6809) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
