## watercolor-rose

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js project that creates a 3D watercolor rose using Three.js, React Three Fiber, and custom shaders. The main feature is a watercolor-style rendered 3D rose with painterly effects achieved through GLSL shaders.

## Key Commands

- `npm run dev` - Start development server with Turbopack
- `npm run build` - Build for production
- `npm start` - Start production server
- `npm run lint` - Run ESLint

## Development Notes

- User typically has dev server running - ask for confirmation of functionality after changes rather than starting new server

## Architecture

### Core Structure
- **Next.js App Router**: Uses the new app directory structure
- **3D Rendering**: React Three Fiber (@react-three/fiber) with Three.js for WebGL rendering
- **UI Components**: React Three Drei (@react-three/drei) for common 3D utilities

### Key Files

- `src/app/components/WatercolorRose.tsx` - Main 3D rose component with Canvas setup
- `src/app/components/PaperEnvironment.tsx` - Camera-responsive paper texture background
- `src/app/components/WatercolorMaterial.tsx` - Custom shader material component
- `src/app/components/Rose.tsx` - GLTF rose model component
- `src/app/shaders/` - Contains .vert/.frag shader files for watercolor effect

### Watercolor Reference Guide

**Primary Tutorial**: https://sinestesia.co/blog/tutorials/watercolor-eevee/

Key techniques for watercolor shader implementation:

1. **Paper Texture Approach**
   - Use window coordinates for consistent paper background
   - Desaturate and adjust contrast with curves
   - Disable paper texture for lighting calculations

2. **Material Building Strategy**
   - Build material "from light to dark"
   - Let paper texture show through light areas
   - Cover shadows and edges to suggest form

3. **Essential Shader Nodes to Recreate in GLSL**
   - Layer Weight node (blend ~0.2) → Fresnel calculations
   - Noise nodes for vector distortion → Procedural noise functions
   - Shader-to-RGB → Lighting calculations
   - ColorRamps with strategic stops → Custom color mapping functions
   - Mix nodes set to "multiply" → Multiply blending mode

4. **Artistic Principles**
   - Create zigzag transitions between colors for organic watercolor bleeding
   - Use soft color ramps (bspline interpolation)
   - Simulate paint "flowing" using screen-space coordinates
   - Goal: "look like painted on paper rather than an object in 3D space"

### Shader System

The watercolor effect is achieved through custom GLSL shaders that:

- Apply painterly normal distortion using noise functions
- Create watercolor-like color transitions with zigzag patterns
- Use Fresnel effects for edge highlighting
- Implement multiply blending to simulate watercolor paint on paper

The shaders are stored as separate .vert/.frag files and imported directly.

### 3D Scene Structure

- GLTF rose model with centered geometry
- Camera-responsive paper texture background
- Auto-rotating OrbitControls for interaction
- Directional and ambient lighting setup
- Shadow mapping enabled

## Key Technologies

- Next.js 15 with App Router
- React 19
- Three.js for 3D rendering
- React Three Fiber for React integration
- TypeScript for type safety
- Tailwind CSS for styling
- ESLint for code quality

---
> Source: [frankievx/watercolor-rose](https://github.com/frankievx/watercolor-rose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
