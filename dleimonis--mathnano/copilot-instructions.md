## mathnano

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Lemon Math is an AI-powered math problem solver that recognizes handwritten math problems from photos and provides step-by-step solutions. The project consists of:

1. **iOS App** (`LemonMath/`) - SwiftUI native app for iOS 17+
2. **Web App** (`web/`) - Vanilla HTML/CSS/JS web version for testing
3. **App Store Assets** (`AppStore/`) - Privacy policy and store preparation materials

## Build & Run Commands

### iOS App (requires macOS with Xcode 15+)
```bash
# Open in Xcode
open LemonMath/LemonMath.xcodeproj

# Build via command line
xcodebuild -scheme LemonMath -destination 'platform=iOS Simulator,name=iPhone 15'

# Run tests
xcodebuild test -scheme LemonMath -destination 'platform=iOS Simulator,name=iPhone 15'

# Build using Swift Package Manager
cd LemonMath && swift build
```

### Web App (cross-platform)
```bash
# Using Python (simplest)
cd web && python3 -m http.server 8000
# Then open http://localhost:8000

# Using Node.js
npx serve web

# Run web tests (validates file structure and JavaScript syntax)
./scripts/test-web.sh      # macOS/Linux
scripts\test-web.bat       # Windows
```

## Architecture

### Core Data Flow
1. User captures/uploads image of handwritten math problem
2. `ImagePreprocessor` normalizes the image (lighting, contrast, noise reduction)
3. `GeminiService` sends image to Google Gemini Vision API with specialized prompts
4. API returns structured JSON: problem text, solution, step-by-step explanation
5. If confidence < 0.7, system retries with alternative preprocessing variants
6. Results displayed with animated handwriting effect in `SolutionView`

### Key Services

- **GeminiService** (`LemonMath/LemonMath/Services/GeminiService.swift`) - Google Gemini Vision API integration with retry logic
- **ImagePreprocessor** (`LemonMath/LemonMath/Utilities/ImagePreprocessor.swift`) - CoreImage pipeline for handwriting normalization
- **APIUsageManager** (`LemonMath/LemonMath/Services/APIUsageManager.swift`) - Rate limiting (10 req/min, 100 req/hour, 500 req/day) and quota tiers (Free: 50/day, Premium: 1000/day, Unlimited)

### API Configuration

- **Model**: `gemini-2.0-flash`
- **Endpoint**: `https://generativelanguage.googleapis.com/v1beta/models/`
- **API key storage**: iOS uses `@AppStorage("geminiAPIKey")`, Web uses `localStorage`
- **Get API key**: https://aistudio.google.com/apikey

### Admin Dashboard Access
Hidden from regular users. Access by long-pressing the Version number in Settings for 3 seconds.

### Localization
English (en) and Greek (el) - files in `LemonMath/LemonMath/Resources/Localizable/`

## CI/CD

GitHub Actions workflow (`.github/workflows/ios-build-test.yml`) runs on:
- Push to `main` or `claude/*` branches
- Pull requests to `main`

The workflow builds the iOS app on macOS 14 runner with Xcode 15.2.

## Development Notes

- iOS: macOS with Xcode 15+, iOS 17.0+ target, Swift 5.9+ with `@Observable`
- Web: Any OS, no build step required
- All API calls use `async/await`
- App Store submission checklist: `AppStore/APP_STORE_CHECKLIST.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dleimonis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
