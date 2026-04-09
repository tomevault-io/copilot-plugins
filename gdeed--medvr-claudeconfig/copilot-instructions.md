## medvr-claudeconfig

> {{APP_DESCRIPTION}} for Apple Vision Pro.

# {{APP_NAME}} - {{APP_DESCRIPTION}}

{{APP_DESCRIPTION}} for Apple Vision Pro.

## Tech Stack
- SwiftUI + RealityKit ({{MIN_PLATFORM}})
<!-- IF:HAND_TRACKING -->
- ARKit (hand tracking)
<!-- END:HAND_TRACKING -->
<!-- IF:IMAGE_TRACKING -->
- ARKit (image tracking)
<!-- END:IMAGE_TRACKING -->
<!-- IF:SHAREPLAY -->
- GroupActivities (SharePlay)
<!-- END:SHAREPLAY -->
- Swift 6 strict concurrency

## Build
```bash
xcodebuild -scheme {{SCHEME_NAME}} -destination 'platform=visionOS Simulator'
xcodebuild test -scheme {{SCHEME_NAME}} -destination 'platform=visionOS Simulator,name=Apple Vision Pro'
```

## Key Concepts
<!-- Define your app's key concepts, roles, or modes here after setup -->
<!-- See specs/PRD.md for full product requirements and feature scope -->

## Critical Rules
- NEVER edit `.pbxproj` files directly — create Swift files and add to Xcode manually
- Use `@Observable` for all state classes, never `ObservableObject`/`@Published`
<!-- IF:ECS -->
- Register all RealityKit components and systems in `App.init()`
<!-- END:ECS -->
<!-- IF:SHAREPLAY -->
- All SharePlay messages must be `Codable` and `Sendable`
<!-- END:SHAREPLAY -->
- Track `ImmersiveSpaceState` to prevent double-opens of immersive spaces
- Keep views under 150 lines; extract subviews as separate structs

## Architecture
- Primary state model owns app state; use `@Observable` with `@MainActor`
<!-- IF:SHAREPLAY -->
- Session controller owns SharePlay state (single source of truth)
<!-- END:SHAREPLAY -->
<!-- IF:ECS -->
- ECS pattern for RealityKit: custom `Component` types + `System` types
<!-- END:ECS -->
- See @.claude/rules/ for path-scoped conventions (architecture, design, testing, views, visionOS)

## Project Documentation
- `specs/PRD.md` — Product requirements, feature scope, and user flows
- `specs/architecture.md` — App-specific architecture, data flow, and file organization
- `specs/DesignStyles.md` — Project design system (overrides defaults in .claude/rules/design-style.md)
- `specs/testing.md` — Test structure, run commands, and naming conventions

> **Always check specs/ before building new features** to stay aligned with project requirements and existing patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GDEED)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GDEED)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
