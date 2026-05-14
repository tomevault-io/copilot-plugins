## compose-multiplatform-boilerplate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Compose Multiplatform boilerplate project using Compose Unstyled. It targets Android, iOS, Desktop (JVM), and Web (JS/WASM) platforms. The project demonstrates a Material-free approach to building UIs with a custom theming system based on design tokens.

## Common Commands

### Development
- **Hot reload development (recommended)**: `./gradlew hotRunJvm`
  - Fastest way to develop - changes to `composeApp/src/commonMain/kotlin` auto-reload
  - Desktop-only, but shares codebase with all platforms

### Running on Different Platforms
- **Desktop (JVM)**: `./gradlew runJvm` or use `desktop` run configuration
- **Web (JS)**: `./gradlew jsBrowserDevelopmentRun` or use `web` run configuration
- **Android**: Use `android` run configuration in IDE
- **iOS**: Open `iosApp` in Xcode

### Build & Test
- **Build all**: `./gradlew build`
- **Clean build**: `./gradlew clean build`
- **Check dependencies**: `./gradlew :composeApp:dependencies`

## Architecture

### Source Structure
All application code lives in `composeApp/src/` with the following source sets:
- `commonMain/kotlin` - Shared code across all platforms (primary development location)
- `androidMain/kotlin` - Android-specific code (Activity setup)
- `iosMain/kotlin` - iOS-specific code (ViewController setup)
- `jvmMain/kotlin` - Desktop-specific code (main entry point)
- `webMain/kotlin` - Web-specific code (main entry point)

### Theming System
The project uses Compose Unstyled's token-based theming (see `Theme.kt:37-72`):
- **Design tokens**: Define theme properties (colors, text styles, shapes) as type-safe tokens
- **Theme definition**: `AppTheme` object in `Theme.kt` contains all design tokens
- **Component theming**: Components access theme via `Theme[property][token]` syntax (e.g., `Theme[colors][primaryColor]`)

Key theme concepts:
- `ThemeProperty`: Groups related tokens (colors, textStyles, shapes)
- `ThemeToken`: Individual design values within a property
- Components are built WITHOUT Material dependencies
- Custom interactive components use `minimumInteractiveComponentSize()`, `focusRing()`, and `outline()` modifiers

### Component Pattern
See `Buttons.kt` for the component pattern:
- Custom composables wrap `com.composeunstyled.Button`
- Use `MutableInteractionSource` for interactive states
- Apply theme tokens for colors, shapes, and spacing
- Use modifier extensions for accessibility (focus rings, minimum sizes)

## Platform Configuration

### Kotlin Multiplatform Targets
- **Android**: Min SDK 24, Target SDK 36, JVM 11
- **iOS**: Frameworks for `iosArm64` and `iosSimulatorArm64` (static framework named `ComposeApp`)
- **JVM**: Desktop application with main class `org.example.project.MainKt`
- **JS/WASM**: Browser targets with webpack dev server

### Dependencies
Core dependencies (all in `commonMain`):
- `compose.runtime`, `compose.foundation`, `compose.ui`
- `com.composables:core` (Compose Unstyled)
- `com.composables:icons-lucide` (Icon library)
- `com.composables:ripple-indication` (Material 3-style ripple)
- `compose.components.resources` (for fonts and assets)

Platform-specific exclusions:
- JVM explicitly excludes Material and Material3 compose dependencies

## Project Namespace
Current namespace: `org.example.project` (see `composeApp/build.gradle.kts:86,90,114,118`)
- When renaming, update Android namespace, applicationId, and desktop package references

## Version Catalog
Dependencies managed via `gradle/libs.versions.toml`:
- Compose Unstyled: 1.46.1
- Kotlin: 2.2.20
- Compose: 1.9.1
- Hot Reload: 1.0.0-rc02

---
> Source: [composablehorizons/compose-multiplatform-boilerplate](https://github.com/composablehorizons/compose-multiplatform-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
