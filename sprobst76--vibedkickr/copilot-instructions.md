## vibedkickr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VibedKickr is a Flutter-based indoor cycling training app that controls smart trainers (Wahoo Kickr Core and other FTMS-compatible devices) via Bluetooth Low Energy. It provides structured interval workouts with real-time power zone tracking, session recording, and comprehensive training analysis.

## Build Commands

```bash
# Install dependencies
flutter pub get

# Generate code (Riverpod, Freezed, JSON serializable, Drift) - required before first run
dart run build_runner build --delete-conflicting-outputs

# Run on specific platform
flutter run -d android
flutter run -d macos
flutter run -d linux

# Lint
flutter analyze

# Run tests
flutter test

# Run a single test file
flutter test test/path/to/test_file.dart

# Build release APK
JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64 flutter build apk --release
```

## Architecture

**Clean Architecture with Feature-Based Organization:**

```
lib/
├── core/                    # Shared Infrastructure
│   ├── ble/                 # BLE Manager, FTMS Service, Simulator
│   ├── database/            # Drift SQLite (tables, DAOs, mappers)
│   ├── gpx/                 # GPX Route Processing
│   ├── services/            # Business Services (TrainingLoad, Records)
│   └── theme/               # App Theme, Zone Colors
├── domain/
│   ├── entities/            # Domain Models (Freezed)
│   └── repositories/        # Repository Interfaces
├── data/
│   └── repositories/        # Repository Implementations
├── features/                # Feature Modules
│   ├── dashboard/           # Main Dashboard with Live Data
│   ├── device_connection/   # BLE Scan & Connect
│   ├── workouts/            # Workout Player & Builder
│   ├── routes/              # GPX Routes
│   ├── history/             # Session History
│   ├── training_load/       # PMC Chart (TSS/CTL/ATL/TSB)
│   ├── comeback/            # Comeback Mode
│   ├── settings/            # App Settings
│   └── debug/               # BLE Diagnostic Tools
├── providers/               # Riverpod State Management
└── routing/                 # GoRouter Navigation
```

**State Management:**
- Riverpod for reactive state management
- Stream-based data flows for BLE device discovery and FTMS trainer data
- Key providers: `bleManagerProvider`, `bleConnectionStateProvider`, `ftmsDataProvider`, `athleteProfileProvider`, `liveTrainingDataProvider`, `activeSessionProvider`

**Code Generation:**
- Freezed for immutable data classes
- Riverpod Generator for providers
- Drift for type-safe SQLite database layer
- JSON Serializable for serialization

## Key Files

| File | Purpose |
|------|---------|
| `lib/providers/providers.dart` | Central Riverpod provider definitions |
| `lib/core/ble/ble_manager.dart` | BLE connection management |
| `lib/core/ble/ftms_service.dart` | FTMS protocol implementation |
| `lib/core/database/app_database.dart` | Drift database setup |
| `lib/routing/app_router.dart` | GoRouter navigation config |
| `lib/domain/entities/` | All domain models |

## Platform Notes

- **Android**: Primary platform, full BLE support
- **macOS/Linux**: BLE supported (Linux requires BlueZ 5.43+)
- **Windows**: BLE requires Windows 10 1803+

## Key Domain Concepts

- **Power Zones**: 7-zone Coggan model calculated from athlete's FTP
- **Heart Rate Zones**: 5-zone model from max heart rate
- **Workouts**: Templated intervals with power targets (absolute watts, FTP percentage, or ranges)
- **FTMS Protocol**: Standard BLE protocol for smart trainers (Service UUID: 0x1826)
- **TSS/CTL/ATL/TSB**: Training Stress Score and Performance Management metrics

## Testing

```bash
# Run all tests
flutter test

# Run specific test
flutter test test/domain/entities/session_stats_test.dart

# Run with coverage
flutter test --coverage
```

## Documentation

- [README.md](README.md) - Project overview and setup
- [ARCHITECTURE.md](ARCHITECTURE.md) - Technical architecture details
- [CHANGELOG.md](CHANGELOG.md) - Version history
- [TODO.md](TODO.md) - Roadmap and planned features
- [CONTRIBUTING.md](CONTRIBUTING.md) - Contribution guidelines

---
> Source: [sprobst76/VibedKickr](https://github.com/sprobst76/VibedKickr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
