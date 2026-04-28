## iris

> Iris is a macOS application that displays a webcam feed in a circular, always-on-top window. This is an **LLM-agent-coded project** where the human developer provides feedback but does not write code directly.

# Agent Development Principles for Iris.app

## Project Overview
Iris is a macOS application that displays a webcam feed in a circular, always-on-top window. This is an **LLM-agent-coded project** where the human developer provides feedback but does not write code directly.

## Core Development Principles

### 1. No Xcode IDE Required
- All development must be possible via command-line tools
- Provide `build.sh` and `run.sh` scripts for easy compilation and launch
- Use `xcodebuild` for building, not Xcode IDE
- Developer should only need: macOS, Command Line Tools, and a terminal

### 2. Design-First Approach
- Every feature has a corresponding `design/*.md` file
- Read the design doc before implementing a feature
- Design docs specify:
  - High-level architecture
  - Key classes and their responsibilities
  - Edge cases and gotchas
  - Testing considerations

### 3. Code Quality Standards

#### Simplicity Over Cleverness
- Write straightforward, readable code
- Avoid premature optimization
- No unnecessary abstractions
- Clear naming over comments

#### Native macOS Patterns
- Use AppKit (not SwiftUI) for precise window control
- Follow Apple's Human Interface Guidelines
- Leverage native APIs (AVFoundation, NSStatusBar, etc.)
- Handle macOS-specific behaviors properly

#### Error Handling
- Gracefully handle camera permissions denial
- Handle no camera available scenario
- Handle window positioning on disconnected displays
- Log errors clearly for debugging

### 4. Incremental Development
- Build features one at a time
- Each feature should be testable independently
- Commit working code frequently
- Don't break existing functionality

### 5. Performance Considerations
- Video rendering should be smooth (60fps target)
- Minimize CPU usage when idle
- Efficient memory usage for video buffers
- Window resizing should be responsive

### 6. User Experience
- No crashes or unexpected behavior
- Smooth animations and transitions
- Clear error messages
- Sensible defaults
- Remember user preferences

## Project Structure

```
iris/
├── AGENTS.md                 # This file
├── README.md                 # User-facing documentation
├── design/                   # Feature design documents
│   ├── 00-project-setup.md
│   ├── 01-camera-management.md
│   ├── 02-circular-window.md
│   ├── 03-menu-bar.md
│   ├── 04-resize-interaction.md
│   └── 05-preferences.md
├── Iris/                     # Xcode project
│   ├── Iris.xcodeproj
│   ├── Iris/                 # Source code
│   │   ├── AppDelegate.swift
│   │   ├── CameraManager.swift
│   │   ├── CircularWindow.swift
│   │   ├── MenuBarController.swift
│   │   └── ...
│   └── Resources/
├── build.sh                  # Build script
├── run.sh                    # Run script
└── clean.sh                  # Clean build artifacts
```

## Implementation Order

1. **Project Setup** - Create basic Xcode project, build scripts
2. **Camera Management** - AVFoundation integration, permissions
3. **Circular Window** - Custom NSWindow with circular clipping
4. **Menu Bar** - NSStatusItem with show/hide toggle
5. **Resize Interaction** - Edge detection and resizing
6. **Preferences** - Settings persistence, camera selection, launch at login

## Testing Strategy

### Manual Testing
- Test on multiple macOS versions (if possible)
- Test with different camera sources
- Test edge cases (no camera, denied permissions, etc.)
- Test multi-monitor scenarios
- Test after system sleep/wake

### What to Test for Each Feature
- Happy path works
- Error cases handled gracefully
- Performance is acceptable
- UI is responsive
- Preferences persist correctly

## Common Pitfalls to Avoid

1. **Window Level Issues** - Use correct `NSWindow.Level` for always-on-top
2. **Camera Session Lifecycle** - Stop session when window hidden to save resources
3. **Circular Clipping** - Ensure proper aspect ratio in circular bounds
4. **Resize Hit Testing** - Edge detection must account for circular shape
5. **Menu Bar Icon** - Use template images for proper dark/light mode support
6. **Memory Leaks** - Properly clean up AVFoundation resources
7. **Thread Safety** - UI updates on main thread, camera on background thread

## Code Style

- **Indentation**: 4 spaces
- **Line Length**: Prefer < 100 characters
- **Naming**:
  - Classes: PascalCase
  - Variables/functions: camelCase
  - Constants: camelCase with `k` prefix or UPPER_SNAKE for globals
- **File Organization**: One class per file (unless tightly coupled)
- **Comments**: Only when code intent isn't obvious

## Decision Log

### Why AppKit over SwiftUI?
- Need precise control over window shape and behavior
- Better documentation and examples for custom window management
- More mature APIs for menu bar applications

### Why Not Use Third-Party Frameworks?
- Keep dependencies minimal
- Native APIs are well-suited for this use case
- Easier to maintain and debug

## Agent Workflow

When implementing a feature:

1. **Read** the corresponding design doc in `design/`
2. **Understand** the requirements and architecture
3. **Implement** following the principles above
4. **Test** manually by running `./run.sh`
5. **Verify** the feature works as designed
6. **Document** any deviations from the design doc

## Notes for Human Developer

- Run `./build.sh` to compile
- Run `./run.sh` to launch the app
- Run `./clean.sh` to clean build artifacts
- Check Console.app for debug logs
- Test camera permissions in System Settings > Privacy & Security > Camera

---
> Source: [ahmetb/Iris](https://github.com/ahmetb/Iris) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
