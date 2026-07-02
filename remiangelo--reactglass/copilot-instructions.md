## reactglass

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is an iOS 26 Liquid Glass React component library that implements Apple's advanced glass simulation system with physics-based rendering. The library provides realistic glass effects using Fresnel equations, chromatic dispersion, caustics, and refraction - matching iOS 26's actual glass rendering rather than simple blur effects.

## Development Commands

```bash
# Install dependencies
npm install

# Development
npm run dev          # Run example app on port 3000 with Vite
npm run storybook    # Interactive component documentation on port 6006

# Build & Quality
npm run build        # Build library with Rollup (outputs to dist/)
npm run lint         # Run ESLint on src/ directory
npm test            # Run Jest tests (currently no tests implemented)

# Build outputs
# - dist/index.js (CommonJS)
# - dist/index.esm.js (ES Modules)
# - dist/index.d.ts (TypeScript declarations)
```

## Architecture

### Advanced Glass Effect System

The library now implements realistic glass physics:

1. **Fresnel Effects** (`src/utils/glassPhysics.ts`)
   - Calculates reflection intensity based on viewing angle
   - Uses Snell's law for accurate light behavior
   - Handles total internal reflection

2. **Chromatic Dispersion**
   - Separate refraction indices for R, G, B channels
   - SVG filters for channel separation
   - Wavelength-dependent IOR calculations

3. **Caustic Light Patterns**
   - Dynamic canvas rendering for light concentration effects
   - Cardioid caustic mathematics
   - Real-time animation based on light angle

4. **Anisotropic Blur**
   - Directional blur for frosted glass appearance
   - View-angle dependent blur calculations

5. **Performance Optimization** (`src/utils/deviceCapabilities.ts`)
   - Three-tier device detection (high/medium/low)
   - Automatic quality adjustment
   - Progressive enhancement approach

### Component Structure

Enhanced pattern with liquid glass:
```tsx
// Basic usage (auto-detects optimal settings)
<GlassContainer intensity="medium">
  Content
</GlassContainer>

// Advanced usage with full effects
<GlassContainer 
  intensity="strong"
  useAdvancedEffects={true}
  mouseTracking={true}
  chromaticAberration={0.02}
  displacementScale={10}
>
  Content
</GlassContainer>
```

### Key Hooks

1. **useGlassEffect** - Enhanced with Fresnel, anisotropic blur, and physics
2. **useLiquidGlass** - Full liquid glass simulation with caustics
3. **useOptimizedGlass** - Automatic performance optimization

### File Organization
```
src/
├── components/      
│   ├── GlassContainer.tsx  # Main container with advanced effects
│   ├── GlassFilters.tsx    # SVG filters for distortion
│   └── ...other components
├── hooks/          
│   ├── useGlassEffect.ts   # Enhanced physics-based effects
│   └── useLiquidGlass.ts   # Full liquid glass simulation
├── utils/
│   ├── glassPhysics.ts     # Physics calculations
│   └── deviceCapabilities.ts # Performance optimization
├── types/          
└── index.ts        
```

### Glass Physics Implementation

The library simulates real glass properties:
- **Index of Refraction (IOR)**: 1.5 for glass
- **Fresnel Equations**: For realistic reflections
- **Chromatic Aberration**: RGB channel separation
- **Surface Normals**: For curved glass effects
- **Thin Film Interference**: Iridescent effects

### Performance Considerations

- **High Tier**: Full WebGL-quality effects with SVG filters
- **Medium Tier**: Optimized CSS with reduced effects
- **Low Tier**: Basic backdrop-filter only
- Automatic detection based on device capabilities
- iOS devices get optimized settings

## Testing Approach
- Jest is configured but no tests are implemented yet
- Test files should use `*.test.tsx` pattern
- Tests are excluded from the TypeScript build

## Important Notes
- Version 2.0.0 introduces liquid glass physics
- The library name is "ios26-glassmorphism-react" (for npm)
- Peer dependencies: React >=16.8.0 (hooks support required)
- Target ES5 for broad compatibility
- **useAdvancedEffects** prop enables full liquid glass simulation
- Performance scales automatically based on device
- All components maintain iOS 26 liquid glass fidelity

---
> Source: [remiangelo/reactGlass](https://github.com/remiangelo/reactGlass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
