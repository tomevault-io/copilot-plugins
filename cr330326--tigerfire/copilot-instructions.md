## tigerfire

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

**TigerTruck (老虎消防车)** is an educational mobile application for preschool children (ages 3-6) focused on fire safety education. The app features "Xiao Huo" (Little Fire), a tiger firefighter character, who guides children through interactive learning scenarios.

### Supported Platforms
- **Android**: Jetpack Compose + Material 3
- **iOS**: SwiftUI (via KMM shared business logic)

### Project Type
Kotlin Multiplatform Mobile (KMM) project maximizing shared logic while maintaining native UI experiences.

---

## Build and Development Commands

### Android Build
```bash
# Build Debug APK
./gradlew :composeApp:assembleDebug

# Install to connected device
./gradlew :composeApp:installDebug

# Run unit tests
./gradlew :composeApp:testDebugUnitTest
```

### iOS Build
```bash
# Open in Xcode
open iosApp/iosApp

# Or build from command line
cd iosApp/iosApp
xcodebuild -scheme TigerFire -configuration Debug build
```

### Test & Debug Scripts
Located in `scripts/` directory:
- `e2e_test.sh` - End-to-end testing
- `run_ui_tests.sh` - UI automation tests
- `test_badge_fix.sh` - Badge system tests
- `test_parent_screen.sh` - Parent mode tests
- `verify_database.sh` - Database verification
- `monitor_badge_realtime.sh` - Real-time badge monitoring

---

## Repository Structure

```
TigerFire/                        # KMM project root
├── composeApp/                   # Shared Kotlin module
│   ├── src/
│   │   ├── commonMain/          # Platform-independent code
│   │   │   ├── kotlin/com/cryallen/tigerfire/
│   │   │   │   ├── domain/      # Domain layer (models, repositories, use cases)
│   │   │   │   ├── data/        # Data layer (repositories, local storage, resources)
│   │   │   │   ├── presentation/# Presentation layer (ViewModels per scene)
│   │   │   │   │   ├── welcome/     # Welcome screen
│   │   │   │   │   ├── map/        # Map navigation
│   │   │   │   │   ├── firestation/# Fire Station scene
│   │   │   │   │   ├── school/     # School scene
│   │   │   │   │   ├── forest/     # Forest scene
│   │   │   │   │   ├── collection/ # Badge collection
│   │   │   │   │   ├── parent/     # Parent mode
│   │   │   │   │   └── common/     # Shared UI components
│   │   │   │   ├── factory/    # ViewModel factories
│   │   │   │   └── ui/theme/   # Theme configuration
│   │   │   └── composeResources/   # Compose resources
│   │   │       └── files/
│   │   │           └── lottie/     # Lottie animations
│   │   ├── androidMain/         # Android-specific code
│   │   │   ├── kotlin/.../ui/     # Compose UI screens
│   │   │   ├── kotlin/.../navigation/ # Navigation setup
│   │   │   ├── kotlin/.../component/  # Platform components
│   │   │   └── assets/
│   │   │       ├── videos/      # MP4 video files
│   │   │       └── audio/       # Audio files
│   │   ├── iosMain/             # iOS-specific code
│   │   │   └── kotlin/.../      # iOS implementations
│   │   └── androidTest/         # Android UI tests
│   └── build.gradle.kts
├── iosApp/                       # iOS app entry point
│   └── iosApp/                  # SwiftUI UI + navigation
├── specs/                        # Project specifications
│   ├── spec.md                  # Feature specifications
│   ├── plan.md                  # Implementation plans
│   └── tasks.md                 # Detailed task breakdown
├── document/                     # Project documentation
│   ├── E2E_TEST_GUIDE.md        # Testing guide
│   ├── UI_AUTOMATION_TEST_GUIDE.md
│   └── *.md                     # Various reports & guides
├── scripts/                      # Build & test scripts
├── constitution.md               # Project constitution (immutable rules)
├── CLAUDE.md                     # This file - AI assistant guidelines
└── README.md                     # Project overview
```

---

## Architecture (Immutable Rules)

This project strictly follows **Clean Architecture** with enforceable boundaries.

### Layer Boundaries (Cannot Be Violated)

| Layer | Location | Responsibility | Constraints |
|-------|----------|----------------|-------------|
| **Domain** | `commonMain/domain/` | Business logic, use cases, entities | Platform-independent, no dependencies on Data/Presentation |
| **Data** | `commonMain/data/` | Repository implementations, storage | Depends only on Domain |
| **Presentation** | `commonMain/presentation/` | ViewModels, state management | Depends on Domain + Data |
| **UI (Android)** | `androidMain/` | Compose screens | No business logic, delegates to ViewModels |
| **UI (iOS)** | `iosApp/` | SwiftUI screens | No business logic, delegates to shared ViewModels |

### Module Structure (Actual Package Layout)
```
com.cryallen.tigerfire
├── domain/
│   ├── model/              # Data models (GameProgress, Badge, SceneStatus)
│   ├── repository/         # Repository interfaces
│   └── usecase/            # Business use cases
├── data/
│   ├── local/              # SQLDelight database
│   ├── repository/         # Repository implementations
│   └── resource/           # Resource path providers
├── presentation/
│   ├── welcome/            # Welcome screen ViewModel
│   ├── map/                # Map navigation ViewModel
│   ├── firestation/        # Fire Station ViewModel
│   ├── school/             # School ViewModel
│   ├── forest/             # Forest ViewModel
│   ├── collection/         # Badge collection ViewModel
│   ├── parent/             # Parent mode ViewModel
│   └── common/             # Shared presentation components
├── factory/                # ViewModel factories
└── ui/theme/               # Compose theme
```

### Platform Constraints
- **commonMain**: No platform-specific APIs (Android/iOS), no UI code, no lifecycle awareness
- **androidMain**: Jetpack Compose only (no XML layouts), use ExoPlayer for video
- **iosMain**: Lightweight platform implementations, use AVPlayer for video

---

## Core Application Features

### Three Learning Scenarios

| Scene | Description | Interaction | Badge Reward |
|-------|-------------|-------------|--------------|
| **Fire Station** (消防站) | Learn about 4 fire equipment devices | Click devices to watch MP4 videos | 4 badges (1 per device, 4 variants each) |
| **School** (学校) | Emergency response narrative | Auto-play 45s animation | 1 badge |
| **Forest** (森林) | Rescue sheep from fire | Drag helicopter, drop ladder | 2 badges (1 per sheep rescued) |

### Key Systems
- **Badge Collection**: 7 base badges with up to 4 color variants each
- **Scene Unlocking**: Progressive (Fire Station → School → Forest)
- **Parental Controls**: Time management (5/10/15/30 min), math verification, usage stats, progress reset
- **Voice Guidance**: Xiao Huo character provides guidance via voice audio

---

## Technology Stack

### Core Technologies
| Technology | Purpose | Platform |
|------------|---------|----------|
| Kotlin Multiplatform Mobile | Shared business logic | All |
| Kotlin Coroutines + Flow | Async operations | All |
| Jetpack Compose | Android UI | Android |
| Material 3 | Design system | Android |
| SwiftUI | iOS UI | iOS |
| SQLDelight | Local database | All |
| Lottie | UI animations | All |
| ExoPlayer | Video playback | Android |
| AVPlayer | Video playback | iOS |
| Kotlinx Serialization | JSON serialization | All |

### Key Dependencies (from build.gradle.kts)
```kotlin
// Android
implementation(libs.lottie.compose)        // Lottie animations
implementation(libs.exoplayer.exoplayer)   // Video playback
implementation(libs.androidx.navigation.compose)

// Common
implementation(libs.kotlinx.coroutines.core)
implementation(libs.kotlinx.serialization.json)
implementation(libs.sqldelight.coroutines.extensions)
```

### Shared State Model
```kotlin
enum class SceneStatus { LOCKED, UNLOCKED, COMPLETED }

data class Badge(
    val id: String,
    val scene: SceneType,
    val variant: Int = 0  // For color variants (0-3)
)

data class GameProgress(
    val fireStation: SceneStatus,
    val school: SceneStatus,
    val forest: SceneStatus,
    val badges: List<Badge>,
    val totalPlayTime: Long
)
```

---

## Development Workflow

### Specs-Driven Development
AI must follow this order:
1. **specs/spec.md**: Understand requirements
2. **specs/plan.md**: Review implementation plans
3. **specs/tasks.md**: Check detailed task breakdown
4. **Implementation**: Code following architectural boundaries

### Conflict Resolution Priority
1. `constitution.md` (highest authority - immutable)
2. `CLAUDE.md` (this file)
3. `specs/spec.md`
4. `specs/plan.md`
5. User instructions (lowest)

### Coding Constraints
- **Shared module**: Platform-independent code only
- **UI layer**: No business logic
- **Kotlin**: Prefer immutable structures, explicit types
- **Naming**: Domain-driven, avoid UI semantics in shared code

---

## Key Constraints and Forbidden Actions

### AI Must NOT
- Violate Clean Architecture boundaries
- Introduce new dependencies without justification
- Modify public APIs without migration plans
- Refactor code unrelated to current requirements
- Make premature optimizations

### AI Must
- Maintain platform independence in shared module
- Use explicit types in public APIs
- Avoid magic numbers and strings
- Ask clarifying questions when specs are unclear

---

## Design Specifications

### Child-Friendly UI Constraints
- **Touch targets**: ≥100pt (≥120pt for main icons)
- **Spacing**: ≥40pt between adjacent elements
- **Text size**: ≥24pt for children's content
- **Single-touch only**: Ignore multi-touch gestures

### Performance Requirements
| Metric | Target | Description |
|--------|--------|-------------|
| Cold start | ≤1.2s | From tap to first screen |
| Lottie animation | ≥30 FPS | Smooth animation playback |
| Single scene memory | ≤120 MB | Memory per scene |
| App size | ≤300 MB | Total app size with resources |

### Color Palette
| Color | Hex | Usage |
|-------|-----|-------|
| Primary Red | `#E63946` | Fire Station, alerts |
| Primary Blue | `#457B9D` | School, water |
| Primary Green | `#2A9D8F` | Forest, safety |
| Accent Yellow | `#F4A261` | Stars, badges |

---

## Resource Management

### Asset Organization
```
composeApp/src/
├── commonMain/composeResources/files/lottie/  # Lottie JSON animations
│   ├── welcome/
│   ├── firestation/
│   ├── school/
│   └── forest/
└── androidMain/assets/
    ├── videos/              # MP4 video files
    │   ├── firestation/     # 4 equipment videos (15s each)
    │   ├── school/          # 1 narrative animation (45s)
    │   ├── forest/          # 2 rescue clips (10s each)
    │   └── celebration.mp4  # Badge collection milestone (20s)
    └── audio/               # Voice guidance files
```

### Video Content
- **Fire Station**: 4 equipment videos (灭火器、消防栓、云梯、水带)
- **School**: 1 narrative animation (45s)
- **Forest**: 2 rescue clips (10s each)
- **Celebration**: 1 badge collection animation (20s)

---

## Testing

### Test Scripts (in `scripts/`)
| Script | Purpose |
|--------|---------|
| `e2e_test.sh` | End-to-end testing |
| `run_ui_tests.sh` | UI automation tests |
| `test_badge_fix.sh` | Badge system tests |
| `test_parent_screen.sh` | Parent mode tests |
| `test_weekly_chart.sh` | Weekly chart tests |
| `verify_database.sh` | Database verification |
| `monitor_badge_realtime.sh` | Real-time monitoring |

### Test Documentation (in `document/`)
- `E2E_TEST_GUIDE.md` - End-to-end testing guide
- `UI_AUTOMATION_TEST_GUIDE.md` - UI testing guide
- `TESTING_CHECKLIST.md` - Testing checklist

### Target User Testing
- Can 3-4 year olds complete tasks without guidance?
- Can 5 year olds complete forest rescue independently?
- Compatibility: Low-end Android (1GB RAM) and iPhone 8

### Regression Testing Areas
- Scene unlock logic synchronization
- Badge collection variant logic
- Parent mode time limit enforcement

---

## Important Documentation Files

| File | Purpose |
|------|---------|
| `constitution.md` | Project constitution with immutable architectural rules |
| `CLAUDE.md` | This file - AI assistant guidelines |
| `README.md` | Project overview and quick start |
| `specs/spec.md` | Complete feature specifications |
| `specs/plan.md` | Implementation plans |
| `specs/tasks.md` | Detailed task breakdown |
| `document/E2E_TEST_GUIDE.md` | End-to-end testing guide |

---

## Communication Style

- **Output**: Structured, clear, concise
- **Lists**: Preferred over long paragraphs
- **Explanations**: Include "why" when proposing solutions
- **Language**: Chinese (as per project context) or English
- **Code comments**: Chinese or English

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cr330326)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cr330326)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
