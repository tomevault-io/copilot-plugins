## seoulmate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SeoulMate is an Android application built with Jetpack Compose that helps connect people in Seoul. The app provides recommendations for meetups, categorized guides (tourism, K-pop, food, etc.), and community features.

**Tech Stack:**
- Kotlin 1.9.0
- Jetpack Compose (Material3)
- Android SDK 24-34
- Gradle 8.2.0

## Development Commands

### Build & Run
```bash
# Build the project
./gradlew build

# Build debug APK
./gradlew assembleDebug

# Install and run on connected device
./gradlew installDebug

# Clean build
./gradlew clean
```

### Testing
```bash
# Run unit tests
./gradlew test

# Run unit tests for debug variant
./gradlew testDebugUnitTest

# Run instrumented tests (requires connected device/emulator)
./gradlew connectedAndroidTest

# Run specific test class
./gradlew test --tests "com.project.seoulmate.ExampleUnitTest"
```

### Code Quality
```bash
# Check for dependency updates
./gradlew dependencyUpdates

# Lint checks
./gradlew lint

# Generate lint report
./gradlew lintDebug
```

## Architecture

### Package Structure
```
com.project.seoulmate/
├── MainActivity.kt              # Entry point, hosts HomeScreen
├── ui/
│   ├── screens/
│   │   └── HomeScreen.kt       # Main screen with bottom nav, categories, recommendations
│   ├── components/             # Reusable UI components
│   │   ├── BottomNavigationBar.kt
│   │   ├── CategoryGrid.kt
│   │   ├── RecommendationSection.kt
│   │   ├── SearchBar.kt
│   │   └── TopBar.kt
│   └── theme/                  # Theme configuration
│       ├── Color.kt
│       ├── Theme.kt
│       └── Type.kt
```

### UI Architecture

**Composition-based Architecture:**
- Single-activity app using `MainActivity`
- Compose UI with `SeoulMateTheme` wrapper
- Bottom navigation with 5 tabs: 홈, 찜, 등록, 쪽지, 프로필
- Home screen divided into 3 sections:
  - Top bar with translation button
  - Middle: Search + Category grid (12 categories in 4x3 grid)
  - Bottom: Recommendation section with horizontal scrolling cards

**State Management:**
- Currently using local `remember { mutableStateOf() }` for UI state
- Bottom nav selection state tracked in `HomeScreen`

**Design System:**
- Primary brand color: `SeoulMatePrimary` (#6C60FD - purple)
- Custom font: SUIT (Regular, Medium, Bold) defined in `BottomNavigationBar.kt`
- Material3 with dynamic color support (Android 12+)
- Rounded corners: 12-28dp depending on component
- Category grid background: #F3F3F3
- Bottom nav: Selected (#6C60FD), Unselected (#929292)

### Key Components

**CategoryGrid:**
- 12 categories in 4x3 lazy grid
- Categories: 관광, K-팝, K-뷰티, 쇼핑, 한식, 카페, 교통 가이드, 숙소/지역, 클래스, 커뮤니티, 전시/공연, 안전/생활
- Each category has icon + text label
- Click handlers use lambda callbacks

**BottomNavigationBar:**
- Sealed class pattern for type-safe navigation items
- 5 items with icons and labels
- Selection state changes tint color
- Currently functional but navigation destinations not implemented

**RecommendationSection:**
- Fixed bottom section with purple background
- Horizontal scrolling cards (LazyRow)
- Placeholder data (needs backend integration)

## Current Development Status

### Implemented
- Home screen UI structure
- Bottom navigation bar (UI only)
- Category grid with 12 categories
- Recommendation section layout
- Basic theme and color system
- SUIT custom font integration

### TODO (Marked in code)
- Translation button functionality (HomeScreen.kt:39)
- Search screen navigation (HomeScreen.kt:45)
- Category detail screens (HomeScreen.kt:52-55)
- Recommendation "See All" screen (HomeScreen.kt:64)
- Bottom navigation routing (currently only state tracking)
- Backend integration for recommendations

## Important Patterns

### Component Structure
Components follow this pattern:
1. Data classes for models (e.g., `Category`, `BottomNavItem`)
2. Main composable with parameters for state and callbacks
3. Preview composables for Android Studio preview

Example:
```kotlin
@Composable
fun MyComponent(
    modifier: Modifier = Modifier,
    onAction: () -> Unit = {}
) {
    // Implementation
}

@Preview
@Composable
fun MyComponentPreview() {
    SeoulMateTheme {
        MyComponent()
    }
}
```

### Navigation Item Pattern
Use sealed classes for type-safe navigation:
```kotlin
sealed class BottomNavItem(val title: String, @DrawableRes val icon: Int) {
    object Home : BottomNavItem("홈", R.drawable.ic_home)
    // ...
}
```

### Theme Usage
Always wrap composables in `SeoulMateTheme` for consistent styling:
```kotlin
setContent {
    SeoulMateTheme {
        Surface(modifier = Modifier.fillMaxSize()) {
            // Content
        }
    }
}
```

## Development Notes

- Package name: `com.project.seoulmate`
- Min SDK: 24 (Android 7.0)
- Target SDK: 34 (Android 14)
- Korean language strings used throughout UI
- Bottom navigation uses custom drawable icons (ensure icons exist in res/drawable)
- SUIT font files must be in res/font/ directory (suit_regular.ttf, suit_bold.ttf)
- Vertical scroll disabled on HomeScreen:34-35 (commented out)
- Layout uses weight-based distribution: main content (0.55f), recommendations (0.45f)

## Adding New Features

### Adding a New Screen
1. Create new file in `ui/screens/`
2. Follow existing pattern with Scaffold + content
3. Add preview composable
4. Wire up navigation in bottom bar or category grid

### Adding a New Component
1. Create file in `ui/components/`
2. Define data models if needed
3. Create main composable with modifier parameter
4. Add preview for visual testing
5. Use existing color scheme and typography

### Adding Navigation
Currently no navigation library implemented. When adding:
- Consider Jetpack Navigation Compose
- Update `BottomNavigationBar` to use NavController
- Create navigation graph with routes
- Wire category clicks to navigation

---
> Source: [faithjua/SeoulMate](https://github.com/faithjua/SeoulMate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
