## swiftly

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Build the app
xcodebuild -scheme swiftly -configuration Debug build

# Build for release
xcodebuild -scheme swiftly -configuration Release build

# Run from Xcode
# Open swiftly.xcodeproj and use Product → Run (⌘R)
```

## Architecture Overview

Swiftly is a macOS menu bar app that provides voice-to-text dictation. The pipeline is:

**Voice → WhisperKit (local transcription) → Optional LLM cleanup → Typed output**

### Core Components

**App Layer** (`swiftly/App/`)
- `AppDelegate.swift` - Main application controller. Manages the menu bar, floating panel, hotkey handling, and orchestrates the transcription pipeline
- `AppState.swift` - Observable state for UI binding (status, onboarding state, session stats)
- `LocalDictateApp.swift` - SwiftUI app entry point with `@NSApplicationDelegateAdaptor`

**Core Services** (`swiftly/Core/`)
- `AudioRecorder.swift` - Captures microphone input, resamples to 16kHz for WhisperKit
- `Transcriber.swift` - Wraps WhisperKit for local speech-to-text (uses `base.en` model)
- `HotkeyManager.swift` - Global hotkey listener using CGEvent tap (Right Option or Right Command to record)
- `KeyboardOutput.swift` - Types text into active app using CGEvent Unicode input
- `ProviderManager.swift` - Routes text cleanup requests to the configured AI provider

**AI Providers** (`swiftly/Core/Providers/`)
- `AIProvider.swift` - Protocol and types (`AIProviderType`, `ProviderResponse`, `AIProviderError`)
- `ClaudeProvider.swift`, `OpenAIProvider.swift`, `GeminiProvider.swift` - API implementations that clean/polish transcribed text

**UI** (`swiftly/UI/`)
- `FloatingPanel.swift` - Borderless window that appears near the notch during recording
- `StatusIndicatorView.swift` - Shows recording/processing status in the floating panel
- `SettingsView.swift` - Provider configuration, API keys, permissions, coding mode
- `AnalyticsCardView.swift` - Usage stats dashboard (words, transcriptions, time saved, API usage)
- `OnboardingView.swift` - First-launch setup flow

**Utilities** (`swiftly/Utilities/`)
- `UserPreferences.swift` - UserDefaults wrapper for settings and analytics stats
- `KeychainManager.swift` - Secure storage for API keys
- `PermissionsManager.swift` - Accessibility and microphone permission checks

### Key Data Flow

1. User holds Right Option → `HotkeyManager` fires callback → `AppDelegate.startRecording()`
2. `AudioRecorder` captures audio at 16kHz
3. User releases key → `AppDelegate.stopRecordingAndProcess()`
4. `Transcriber.transcribe()` runs WhisperKit locally
5. If API key configured, `ProviderManager.clean()` sends text to AI provider
6. `KeyboardOutput.type()` injects text into active app via CGEvent
7. Stats recorded to `UserPreferences`

### Dependencies

- **WhisperKit** - Local speech recognition (downloads model on first launch)
- macOS 14+ (Sonoma) required
- Requires Accessibility and Microphone permissions

### Settings Storage

- API keys: macOS Keychain (via `KeychainManager`)
- User preferences: UserDefaults (via `UserPreferences`)
- Analytics stats persist across sessions in UserDefaults

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/luanmdang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
