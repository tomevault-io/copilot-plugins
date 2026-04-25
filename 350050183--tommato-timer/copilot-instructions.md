## tommato-timer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Flutter Development
```bash
# Install dependencies
flutter pub get

# Run the app in development mode
flutter run

# Build for specific platforms
flutter build apk          # Android APK
flutter build ios          # iOS build
flutter build web          # Web build

# Code analysis and linting
flutter analyze            # Static analysis using analysis_options.yaml

# Generate launcher icons
flutter pub run flutter_launcher_icons:main

# Clean build artifacts
flutter clean
```

### Testing
```bash
# Run all tests
flutter test

# Run widget tests specifically
flutter test test/widget_test.dart
```

## Project Architecture

### Core Architecture Pattern
This is a Flutter app implementing a Pomodoro timer with 3D cube interface using **Provider** pattern for state management:

- **main.dart**: Entry point with MultiProvider setup for dependency injection
- **TimerProvider**: Main business logic controller managing timer state, notifications, and audio
- **TimerModel**: Core timer data model with state transitions and history tracking  
- **SettingsModel**: User preferences and configuration management

### State Management Flow
```
SettingsModel → TimerModel → TimerProvider → UI Widgets
```
- Settings flow down through providers using ChangeNotifierProxyProvider
- UI updates automatically via Consumer widgets listening to provider changes
- Timer state persisted using SharedPreferences for background recovery

### Key Components

#### 3D Cube Timer Interface (`lib/widgets/three_d_cube.dart`)
- Uses WebView + Three.js for 3D cube rendering on mobile platforms
- Each cube face represents different timer durations (5, 10, 15, 20, 25, 30 minutes)
- JavaScript-Flutter communication via WebView channels for face selection
- 3D model files located at `assets/cube/tomato-timer.obj` and `assets/cube/tomato-timer.mtl`
- **Important**: Web platform shows placeholder - 3D cube only works on mobile

#### Timer Logic (`lib/models/timer_model.dart`)
- Implements three timer types: work (25min), short break (5min), long break (15min)
- Automatic progression: work → short break → work → ... → long break (every 4 cycles)
- Timer history tracking with actual vs planned duration
- State machine: initial → running → paused → finished

#### Background Notifications (`lib/providers/timer_provider.dart`)
- Platform-specific background handling:
  - **iOS**: Uses local scheduled notifications for background timer completion
  - **Android**: Uses Workmanager for background tasks
- Audio completion notifications using audioplayers package
- Notification permissions handled automatically

#### Internationalization
- Supports Chinese (zh) and English (en) locales
- Custom localization implementation in `lib/utils/l10n/`
- Locale selection based on system settings with override option

### Platform-Specific Notes

#### Mobile Platforms (iOS/Android)
- Full 3D cube functionality with touch interaction
- Background timer notifications work properly
- Audio completion sounds supported
- Vibration feedback available

#### Web Platform
- 3D cube shows placeholder message ("3D 立方体在 Web 平台暂不可用")
- Background notifications limited by browser constraints
- WebView JavaScript methods not available on web

### File Organization
```
lib/
├── main.dart                    # App entry point & provider setup
├── models/                      # Data models
│   ├── timer_model.dart        # Core timer logic & state
│   ├── timer.dart              # Timer data structures
│   └── settings.dart           # User settings model
├── providers/                   # State management
│   └── timer_provider.dart     # Main business logic controller
├── screens/                     # Full-screen pages
├── widgets/                     # Reusable UI components
│   └── three_d_cube.dart      # 3D timer interface
├── services/                    # External service integrations
│   └── notification_service.dart
└── utils/                       # Utilities & configuration
    ├── l10n/                   # Internationalization
    └── routes.dart             # Navigation routes
```

### Development Guidelines

#### Working with the 3D Cube
- Model files are at `assets/cube/tomato-timer.obj` and `assets/cube/tomato-timer.mtl` - do NOT create `assets/model/`
- Face duration mapping: A=5min, B=10min, C=15min, D=20min, E=25min, F=30min
- JavaScript-Flutter communication uses WebView message channels
- Test 3D functionality only on mobile platforms

#### Timer State Management
- Always use TimerProvider methods (startTimer, pauseTimer, resetTimer) rather than directly modifying TimerModel
- Timer state automatically persists to SharedPreferences for background recovery
- Background notifications are platform-specific - test on actual devices

#### Adding New Features
- Follow Provider pattern - create/update models first, then providers, then UI
- Use existing widgets (glassmorphic_container, timer_display) for consistent UI
- Maintain internationalization by adding strings to both zh.dart and en.dart

#### Asset Management
- Audio files in `assets/sounds/` (complete.mp3 used for timer completion)
- Images in `assets/images/` (app icons and logos)
- 3D models in `assets/cube/` (tomato-timer.obj, tomato-timer.mtl)

### Dependencies Note
This project uses specific dependency overrides in pubspec.yaml due to compatibility issues. When adding new dependencies, check for conflicts with existing overrides in the `dependency_overrides` section.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/350050183) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
