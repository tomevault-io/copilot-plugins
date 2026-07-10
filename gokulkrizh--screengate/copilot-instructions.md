## screengate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ScreenGate is an iOS SwiftUI application that implements a digital wellness and screen time management system. The app features a comprehensive 19-step onboarding flow, personalized insights, and tools to help users develop healthier phone usage habits. It serves as a screening/filtering tool with a clean MVVM architecture.

## Architecture

The project follows **MVVM (Model-View-ViewModel)** architecture:

- **Models** (`screengate/Models/`): Data structures following Swift best practices
- **ViewModels** (`screengate/ViewModels/`): Business logic with `@MainActor` and `@Published` properties for reactive UI updates
- **Views** (`screengate/Views/`): SwiftUI views with clear separation of concerns
  - **Onboarding Views** (`Views/Onboarding/`): Comprehensive 19-step guided setup flow
  - **Main Views**: Splash screen, main navigation, and content views
- **Main App**: Entry point in `screengateApp.swift` using SwiftUI's `@main` attribute

### Key Architectural Patterns
- SwiftUI navigation with progressive onboarding flow
- Reactive state management using Combine framework
- Thread safety with `@MainActor` annotation on ViewModels
- Component-based UI with reusable onboarding components
- Persistent onboarding state tracking with UserDefaults
- Animated splash screen with smooth transitions

## Build and Development Commands

### Building the Project
```bash
# Build for iOS Simulator (recommended for development)
xcodebuild -scheme screengate -configuration Debug -destination 'platform=iOS Simulator,name=iPhone 17 Pro'

# Build for device
xcodebuild -scheme screengate -configuration Debug
```

### Running the Application
```bash
# Install and run on simulator
xcodebuild -scheme screengate -configuration Debug -destination 'platform=iOS Simulator,name=iPhone 17 Pro' build
xcrun simctl install booted ./DerivedData/Build/Products/Debug-iphonesimulator/screengate.app
xcrun simctl launch booted com.gia.screengate
```

### Available Schemes and Configurations
- **Scheme**: `screengate`
- **Configurations**: `Debug`, `Release`
- **Target**: `screengate`
- **Bundle Identifier**: `com.gia.screengate`

## Project Structure

```
screengate/
├── Models/                    # Data models (ContentModel, OnboardingData)
├── ViewModels/                # Business logic and state management
│   ├── ContentViewModel.swift # Main content logic
│   └── OnboardingViewModel.swift # 19-step onboarding flow management
├── Views/                     # SwiftUI views and components
│   ├── SplashView.swift       # Animated splash screen (2.5s duration)
│   ├── MainView.swift         # Main navigation controller
│   ├── ContentView.swift      # Primary app content
│   └── Onboarding/            # Onboarding flow components
│       ├── OnboardingView.swift          # Main onboarding container
│       ├── OnboardingComponents.swift    # Reusable UI components
│       └── OnboardingStepViews.swift     # Individual step implementations
├── Assets.xcassets/           # App icons, colors, and visual assets
└── screengateApp.swift        # App entry point
```

## Development Notes

### Data Flow
- `ContentViewModel` manages `@Published` properties for reactive UI updates
- `OnboardingViewModel` handles the 19-step onboarding flow with persistent state
- Views use `@StateObject` to maintain ViewModel lifecycle
- UI actions call ViewModel methods, which update state and trigger view refreshes
- Onboarding progress is tracked and persisted using UserDefaults

### App Launch Flow
1. **Splash Screen** (2.5 seconds): Animated ScreenGate logo with door icon
2. **Onboarding Check**: Determines if user has completed onboarding
3. **Main Content**: Either shows onboarding flow or main app interface

### Onboarding Flow (19 Steps)
- Welcome and introduction
- Daily screen time assessment
- Problem habits identification
- Personal goal setting
- 6-question behavioral survey
- Age and occupation collection
- Screen Time permission request
- Personalized projection display
- App selection and customization
- Mindful pause setup
- Scheduling preferences
- Gamification options
- Social connection settings
- Notification preferences
- Completion

### Current Limitations
- No data persistence beyond onboarding state (onboarding data stored in UserDefaults)
- No networking or external data sources
- No testing infrastructure implemented
- Mock Screen Time API integration
- Simplified app selection (placeholder implementation)

### Code Style Requirements
- Use `@MainActor` for ViewModels to ensure UI thread safety
- Follow SwiftUI best practices for view composition
- Implement proper error handling and loading states
- Use `@Published` properties for reactive state management
- Maintain clear separation between onboarding and main app logic
- Use reusable components for onboarding UI elements
- Implement smooth animations and transitions between steps

## Dependencies

- **iOS SDK**: 18.0+ (for modern SwiftUI features)
- **Swift Version**: 5.0
- **Frameworks**: SwiftUI, Foundation, Combine
- **No external dependencies**: Uses only native iOS frameworks
- **Screen Time API**: Integration planned for digital wellness features

## Key Features Implemented

### ✅ Completed Features
- **Splash Screen**: Animated 2.5-second introduction with door icon
- **Main Navigation**: Flow control between splash, onboarding, and main app
- **Comprehensive Onboarding**: 19-step guided setup flow
- **Progress Tracking**: Visual progress bar and step indicators
- **Data Collection**: User preferences, habits, and behavioral survey
- **State Persistence**: Onboarding completion status saved to UserDefaults
- **Component Architecture**: Reusable onboarding UI components

### 🚧 Planned Features
- Screen Time API integration
- App usage tracking and limits
- Mindful pause interventions
- Gamification system (XP, streaks, achievements)
- Social features and progress sharing
- Analytics and insights dashboard
- Notification system
- Data persistence for user settings and progress

## Common Development Patterns

When adding new features:
1. Define data models in `Models/` directory
2. Create/extend ViewModels in `ViewModels/` directory with `@MainActor`
3. Implement UI components in appropriate `Views/` subdirectory
4. For onboarding features, extend existing flow in `Views/Onboarding/`
5. Ensure proper error handling and loading states
6. Maintain MVVM separation (no business logic in Views)
7. Use `@Published` properties for reactive UI updates
8. Consider user experience with smooth animations and transitions

---
> Source: [gokulkrizh/screengate](https://github.com/gokulkrizh/screengate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
