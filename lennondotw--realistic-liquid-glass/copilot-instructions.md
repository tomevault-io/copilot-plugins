## realistic-liquid-glass

> - Always follow the CODE_OF_CONDUCT.md in the root of the project.

# Project Level Claude Memories

- Always follow the CODE_OF_CONDUCT.md in the root of the project.
- **CRITICAL**: Always use English for ALL code, comments, documentation, memories, commit messages, and any written content in this project. Only use other languages for conversational communication.

## Project Overview

This is a modern React component library focused on implementing Apple's iOS 26 Liquid Glass effect as high-quality React components. The project follows strict TypeScript development standards and modern frontend toolchain.

## Liquid Glass Design Analysis

### Apple WWDC 2025 Background

Apple introduced "Liquid Glass" design language at WWDC 2025 for iOS 26, marking the biggest design revolution since iOS 7. Liquid Glass is a complex digital meta-material with characteristics:

- Translucent material that reflects and refracts surrounding content
- Real-time specular highlights responding to user actions
- Adaptive to light/dark environments with color influenced by surrounding content
- Fluid morphing with dynamic transitions between controls

### Technical Implementation Strategy

Based on analysis, determined technical approach:

1. **SVG Filter System**: Displacement mapping + chromatic aberration
2. **Background Blur**: `backdrop-filter` + saturation adjustment
3. **Dynamic Interaction**: Mouse-aware elastic deformation
4. **Edge Highlights**: Dynamic gradient borders
5. **Compatibility Handling**: Firefox/Safari fallback strategies

### Web Platform Limitations

- Cannot achieve complete 1:1 reproduction (~90% similarity achievable)
- Lacks real environment mapping
- No deep system-level graphics API integration
- Limited hardware acceleration capabilities

## Styling Requirements

### Tailwind CSS Prefix

- **CRITICAL**: Always use `rlg:` prefix for all Tailwind classes to prevent style pollution
- Example: `rlg:flex rlg:items-center rlg:px-4`
- Never use unprefixed classes like `flex items-center px-4`

## Component Implementation Status

### Current State

- LiquidGlass component is in WIP state (shows "🚧 WIP LiquidGlass")
- Basic project structure and tooling setup complete
- Storybook configuration ready

### Next Steps

- Implement high-quality LiquidGlass component combining both reference demo advantages
- Focus on SVG filter techniques and physical deformation algorithms
- Ensure good compatibility and performance
- Complete Storybook demos and test coverage

This project represents modern React component library development best practices, combining strict type safety, modern toolchain, and high-quality component design goals.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lennondotw)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lennondotw)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
