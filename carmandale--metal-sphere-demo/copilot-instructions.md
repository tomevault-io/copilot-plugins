## metal-sphere-demo

> This project follows a clear organizational structure for visionOS development.

# Project Structure

This project follows a clear organizational structure for visionOS development.

## Directory Organization
- `MetalSphereDemo/` - Main project directory
  - `Compute/` - Metal compute systems and utilities
    - [ComputeSystem.swift](mdc:MetalSphereDemo/Compute/ComputeSystem.swift) - Base class for compute systems
    - [ComputeUtilities.swift](mdc:MetalSphereDemo/Compute/ComputeUtilities.swift) - Utility functions
    - [SphereFractal.metal](mdc:MetalSphereDemo/Compute/SphereFractal.metal) - Compute shader for fractal generation
  - `Shaders/` - Metal shader code
    - [SphereParams.h](mdc:MetalSphereDemo/Shaders/SphereParams.h) - Shader parameters header
    - [SphereShader.metal](mdc:MetalSphereDemo/Shaders/SphereShader.metal) - Main shader code

## Component Separation
- Separate immersive experiences, volumes, and standard SwiftUI views into distinct files
- Keep Metal shaders in the dedicated `Shaders` directory
- Store compute systems in the `Compute` directory for clear separation of concerns
- Use Swift packages for reusable components

## File Naming Conventions
- Use descriptive names that indicate functionality
- Suffix Swift files with their primary type (View, System, etc.)
- Suffix Metal shader files with `.metal`
- Use `.h` for Metal header files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carmandale)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/carmandale)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
