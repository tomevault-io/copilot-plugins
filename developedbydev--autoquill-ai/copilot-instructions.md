## autoquill-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Build and Run Commands
```bash
# Clean build environment
flutter clean

# Get dependencies
flutter pub get

# Install macOS CocoaPods dependencies
cd macos && pod install && cd ..

# Run in debug mode
flutter run -d macos

# Build for release
flutter build macos --release

# Generate app icons
flutter pub run flutter_launcher_icons

# Run linting
flutter analyze

# Format code
flutter format .
```

### Testing Commands
```bash
# Run Dart tests
flutter test

# Note: This project primarily uses manual testing and integration testing
# No specific test running commands beyond standard Flutter test
```

### Distribution Commands
```bash
# Complete build, sign, and notarize for distribution
./scripts/build_and_sign.sh

# Build DMG installer
./scripts/build_dmg_complete.sh

# Output: dist/signed/{timestamp}/AutoQuill-v{version}-notarized.zip
# Output: dist/signed/{timestamp}/AutoQuill-v{version}.dmg
```

## Architecture Overview

### High-Level Architecture
AutoQuill AI follows **Clean Architecture** with **BLoC pattern** for state management:

- **Presentation Layer**: Flutter widgets with BLoC for state management
- **Domain Layer**: Business logic and repository interfaces
- **Data Layer**: API clients, local storage (Hive), and platform channels
- **Platform Layer**: macOS-specific Swift code for system integration

### Key Components

#### Core Services
- **HotkeyHandler** (`lib/features/hotkeys/core/hotkey_handler.dart`) - Centralized global hotkey management
- **AssistantService** (`lib/features/assistant/assistant_service.dart`) - AI-powered text processing
- **SmartTranscriptionService** (`lib/features/transcription/services/smart_transcription_service.dart`) - Groq API integration
- **SettingsService** (`lib/core/settings/settings_service.dart`) - Centralized settings management
- **AppStorage** (`lib/core/storage/app_storage.dart`) - Hive-based local storage wrapper

#### Feature Modules
- **Transcription** (`lib/features/transcription/`) - Speech-to-text functionality
- **Recording** (`lib/features/recording/`) - Audio recording with overlay UI
- **Hotkeys** (`lib/features/hotkeys/`) - Global hotkey registration and handling
- **Assistant** (`lib/features/assistant/`) - AI assistant mode for text enhancement
- **Settings** (`lib/features/settings/`) - Application configuration UI
- **Onboarding** (`lib/features/onboarding/`) - First-time user setup flow

### State Management
- **BLoC Pattern**: All features use BLoC for state management
- **Repository Pattern**: Clean separation between data sources and business logic
- **Dependency Injection**: GetIt for service location (`lib/core/di/injection_container.dart`)

### Storage
- **Hive Database**: Local storage for settings, transcriptions, and app state
- **AppStorage Wrapper**: Simplified interface over Hive boxes
- **Key Storage**: Secure API key storage and retrieval

### Platform Integration
- **macOS Swift Code**: System-level integration in `macos/Runner/`
- **Recording Overlay**: Native macOS overlay window for recording feedback
- **Hotkey Manager**: System-wide hotkey registration using `hotkey_manager` package
- **WhisperKit**: Local AI transcription models (optional)

## Development Guidelines

### State Management Patterns
- Use BLoC for all stateful components
- Repository pattern for data access
- Events and states should be immutable
- Use Equatable for value equality

### Hotkey System
- All hotkeys managed through `HotkeyHandler` singleton
- Hotkey registration happens after UI render for performance
- Escape key always cancels active recordings
- Conflict detection prevents duplicate hotkey registration

### API Integration
- Groq API for transcription and AI assistance
- HTTP client reuse for performance
- Error handling with user-friendly messages
- Rate limiting considerations

### Local Storage
- Hive for all persistent data
- AppStorage wrapper for simplified access
- Settings use ValueListenable for reactive updates
- Proper box closing on app shutdown

### UI/UX Guidelines
- Minimalist design with clean typography (Inter font)
- Dark/light theme support
- Non-intrusive overlays for recording feedback
- Responsive layout for different window sizes

## Key Configuration Files

### Flutter Configuration
- `pubspec.yaml` - Dependencies and app metadata
- `analysis_options.yaml` - Linting rules (uses `flutter_lints`)

### macOS Configuration
- `macos/Runner/Info.plist` - App permissions and metadata
- `macos/Runner/Release.entitlements` - App capabilities for release builds
- `macos/Runner/DebugProfile.entitlements` - Debug/profile capabilities

### Build Configuration
- `scripts/build_and_sign.sh` - Production build script with code signing
- `scripts/build_dmg_complete.sh` - DMG installer creation
- `macos/Podfile` - CocoaPods dependencies

## Important Notes

### Development Environment
- Requires macOS for development and testing
- Xcode required for iOS/macOS builds
- Flutter SDK 3.1.3+ required
- CocoaPods required for macOS dependencies

### Testing Strategy
- Manual testing is primary approach
- Integration testing for critical paths
- Test hotkey functionality across different apps
- Verify API key management and security

### Performance Considerations
- Hotkey registration is lazy-loaded after UI render
- HTTP client reuse for API calls
- Sound player pre-initialization
- Efficient BLoC state updates

### Security
- API keys stored securely in Hive
- No sensitive data in version control
- Proper entitlements for macOS capabilities
- Code signing required for distribution

## Distribution Process

### Requirements
- Apple Developer Account ($99/year)
- Developer ID Certificate for code signing
- Notarization profile for Gatekeeper
- App-specific password for notarization

### Build Process
1. Run `./scripts/build_and_sign.sh`
2. Script handles building, signing, and notarization
3. Creates distribution files in `dist/signed/{timestamp}/`
4. Test on clean macOS system before release

### Auto-Updates
- Sparkle framework for auto-updates
- Signed ZIP files for update distribution
- Appcast XML for version management
- Auto-update detection on app startup

---
> Source: [DevelopedByDev/autoquill_ai](https://github.com/DevelopedByDev/autoquill_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
