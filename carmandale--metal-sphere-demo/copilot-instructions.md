## metal-sphere-demo

> This project follows Apple's latest best practices for visionOS 2 development.

# visionOS 2 Best Practices

This project follows Apple's latest best practices for visionOS 2 development.

## Key Files and Components
- [ImmersiveView.swift](mdc:MetalSphereDemo/ImmersiveView.swift) - Main immersive experience
- [FractalSystem.swift](mdc:MetalSphereDemo/FractalSystem.swift) - Custom compute system

## Important Guidelines

### RealityKit Integration
- Use `RealityView` for 3D content integration with SwiftUI
- Initialize heavy resources once in `init()` rather than recreating each frame
- Register compute systems with `ComputeSystemComponent` for efficient GPU processing
- Use `LowLevelTexture` for direct GPU texture access when needed

### SwiftUI Patterns
- Use `@MainActor` for view structs interacting with RealityKit
- Use `@Observable` macro for modern state management
- Implement the latest `.onChange(of:)` modifier with both old and new values
- Keep the update closure in RealityView minimal; leverage systems for per-frame updates
- Use `.ornament()` for floating windows and `.glassBackgroundEffect()` for UI blending

### Asset Management
- Use USDC format for 3D models in Reality Composer Pro
- Employ Reality Composer Pro for scene composition and asset management

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carmandale) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
