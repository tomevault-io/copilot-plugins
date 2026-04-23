## androidaichatclientbasic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AndroidAIChatClientBasic is a native Android chat client that integrates with OpenAI's API for AI-powered conversations. The app is built with:
- Kotlin
- Jetpack Compose for UI
- Hilt for dependency injection
- Coroutines and Flow for async operations
- DataStore for local preferences
- OkHttp for networking with SSE (Server-Sent Events) streaming

## Build Commands

### Building the project
```bash
./gradlew build
```

### Running tests
```bash
# Unit tests
./gradlew test

# Instrumented tests (requires emulator or device)
./gradlew connectedAndroidTest
```

### Cleaning build artifacts
```bash
./gradlew clean
```

### Installing on device/emulator
```bash
./gradlew installDebug
```

### Creating release build
```bash
./gradlew assembleRelease
```

## Architecture

### Dependency Injection
The app uses Hilt for dependency injection. Key modules:
- `NetworkModule` (di/NetworkModule.kt): Provides OkHttpClient and Json serializer with custom timeouts (30s connect, 60s read/write)
- `DataStoreModule` (di/DataStoreModule.kt): Provides DataStore repository for persistent storage

The application class is `MyApplication` annotated with `@HiltAndroidApp`.

### Navigation
Navigation is managed by Jetpack Compose Navigation with type-safe routes using Kotlinx Serialization:
- `NavHome`: Main chat screen
- `NavSettingApiKey`: API key configuration screen

Navigation setup is in `MyAppNavigation.kt`.

### Data Layer

**Repository Pattern:**
- `OpenAIRepository`: Handles OpenAI API communication with streaming support using SSE (Server-Sent Events)
  - `streamChat()` returns a Flow<String> for streaming responses
  - Retrieves API key from DataStore before making requests
  - Uses callbackFlow to convert SSE events to Flow emissions

**Data Models:**
- `ChatRequest`, `ChatMessage`, `ChatStreamResponse`, `ChatStreamChoice`, `ChatDelta` (data/model/ChatModels.kt): OpenAI API request/response models
- `ChatHistory`, `ChatMessage` (data/ChatHistory.kt): Local chat history models

**Persistence:**
- `DataStoreRepository` interface with `PreferencesDataStoreImpl`: Stores API key securely
- Exposes `apiKeyFlow: Flow<String?>` for reactive API key observation

### UI Layer

**Screens:**
- `HomeScreen` (ui/home/HomeScreen.kt): Main chat interface with modal drawer navigation
  - Includes text input field and send button
  - Chat history display (currently placeholder)

- `SettingApiKeyScreen`: API key configuration with password masking
  - Uses `SettingApiKeyViewModel` for state management
  - Saves API key changes immediately to DataStore

### Streaming Implementation

The OpenAI integration uses Server-Sent Events (SSE) for streaming chat responses:
1. Request is made with `stream: true` in ChatRequest
2. `EventSourceListener` processes incoming SSE events
3. Each content chunk is emitted through Flow
4. Stream closes on "[DONE]" message or finishReason

## Key Configuration

- **Package name**: `org.nunocky.androidaichatclientbasic`
- **Min SDK**: 28
- **Target SDK**: 36
- **Compile SDK**: 36
- **Java version**: 11
- **Internet permission**: Required in AndroidManifest.xml

## Dependencies

Key libraries:
- Hilt (DI)
- Jetpack Compose with Material 3
- Navigation Compose with type-safe routes
- Kotlinx Serialization for JSON
- Retrofit (included but not actively used - OpenAI uses direct OkHttp)
- DataStore Preferences
- Material Icons Extended

## Development Notes

- The app stores the OpenAI API key in DataStore Preferences - never commit actual API keys
- OkHttp client is configured with generous timeouts for streaming responses
- Serialization ignores unknown keys and coerces input values for API flexibility
- The repository layer returns Flow for streaming, making it easy to collect in ViewModels/Composables

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nunocky) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
