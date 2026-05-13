## neubrutalism-compose-multiplatform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**NeuBrutalism UI** - A Compose Multiplatform UI component library with NeuBrutalism design style, published via JitPack.

### NeuBrutalism Design Characteristics
- Black borders (1dp default via `NeuShadowDefaults.BorderWidth`)
- Solid offset shadows (6dp x, 4dp y, no blur)
- High contrast, saturated colors
- Bold typography (Space Grotesk)
- Press animations (shadow reduces to 1dp offset + translation)

## Build Commands

```bash
# Build library
./gradlew :neubrutalism:build

# Build demo app (Android debug)
./gradlew :composeApp:assembleDebug

# Build demo app (Android release)
./gradlew :composeApp:assembleRelease

# Run tests
./gradlew :neubrutalism:test

# Publish to local Maven (for testing)
./gradlew :neubrutalism:publishToMavenLocal

# iOS build - open iosApp/ in Xcode and run from there
```

## Architecture

Kotlin Multiplatform library with a demo catalog app.

```
NeuBrutalismUI/
├── neubrutalism/                    # Library module (publishable)
│   └── src/commonMain/kotlin/com/itdeveapps/neubrutalism/
│       ├── components/              # UI components
│       │   ├── NeuButton.kt
│       │   ├── NeuIconButton.kt
│       │   ├── NeuTextField.kt
│       │   ├── NeuSelectionControls.kt
│       │   ├── NeuContainers.kt
│       │   ├── NeuNavigation.kt
│       │   ├── NeuProgress.kt
│       │   ├── NeuSlider.kt
│       │   ├── NeuTab.kt
│       │   ├── NeuDropdown.kt
│       │   └── NeuCalendarDay.kt
│       └── theme/                   # Theme configuration
│           ├── Color.kt
│           ├── NeuBrutalismModifiers.kt
│           ├── Shape.kt
│           ├── Theme.kt
│           └── Typography.kt
├── composeApp/                      # Demo catalog app
│   └── src/commonMain/kotlin/com/itdeveapps/neubrutalism/
│       ├── App.kt                   # Main app with navigation
│       └── catalog/                 # Demo screens
│           ├── CatalogHome.kt
│           └── CategoryScreens.kt
├── iosApp/                          # iOS app wrapper
├── jitpack.yml                      # JitPack build config
└── settings.gradle.kts              # Module includes
```

## Component Categories

| Category | Components |
|----------|------------|
| Buttons | NeuButton (PRIMARY, ACCENT, SUCCESS, ERROR), NeuIconButton |
| Inputs | NeuTextField, NeuSearchField, NeuPasswordField |
| Selection | NeuCheckbox, NeuRadioButton, NeuSwitch |
| Navigation | NeuTopAppBar, NeuBottomNavigation, NeuTab |
| Containers | NeuCard, NeuAlertDialog, NeuBottomSheet |
| Feedback | NeuProgressBar, NeuCircularProgress, NeuSlider, NeuRangeSlider |
| Dropdowns | NeuDropdown, NeuMonthSelector |

## Color Palette

```kotlin
Primary = Yellow (#F4D738)   // Main CTAs, selected states
Accent = Blue (#2563EB)      // Calendar selection, links
Success = Green (#22C55E)    // Confirmations, checkmarks
Error = Red (#EF4444)        // Errors, destructive actions
Surface = White (#FFFFFF)    // Cards, inputs
Background = Cream (#FFFFF5) // App background
Border = Black (#000000)     // All borders and shadows
```

## Key Modifiers

- `neuShadow()` - Solid offset shadow using Compose 1.9 dropShadow API
- `neuBorder()` - Black border wrapper
- `neuPressed()` - Press state translation (moves component toward shadow)
- `neuStyle()` - Combined shadow + border modifier
- `neuPartialBorder()` - 3-sided border for tabs (no bottom)

## Theme Access

```kotlin
// Via CompositionLocal
val colors = NeuBrutalismTheme.colors

// Direct access to color constants
NeuColors.Primary  // Yellow (#F4D738)
NeuColors.Accent   // Blue (#2563EB)
```

## Publishing

Library is published via JitPack. To release a new version:

```bash
git tag 1.0.0
git push origin main --tags
```

Users add the dependency:
```kotlin
// settings.gradle.kts
maven("https://jitpack.io")

// build.gradle.kts
implementation("com.github.tamtom:NeuBrutalismUI:1.0.0")
```

## Tech Stack

- Kotlin 2.3.0
- Compose Multiplatform 1.9.3
- Android SDK 36 (minSdk 24, targetSdk 36)
- iOS (arm64 and simulator arm64)

---
> Source: [tamtom/neubrutalism-compose-multiplatform](https://github.com/tamtom/neubrutalism-compose-multiplatform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
