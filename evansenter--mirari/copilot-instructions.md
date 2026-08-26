## mirari

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mirari is an iOS app for Magic: The Gathering card recognition and collection management. It uses Gemini AI to identify cards by their artwork (not just text), looks them up on Scryfall, and manages a personal collection.

**Key Differentiator**: Uses card art to identify specific printings/versions, unlike existing apps that rely on OCR text matching.

## Technology Stack

- **UI**: SwiftUI
- **Data**: SwiftData
- **Camera**: AVFoundation
- **AI**: Gemini 3 Flash Preview (frame-by-frame) + Gemini 2.5 Flash Live API (streaming)
- **Cards**: Scryfall API
- **Min iOS**: 17.0 (required for SwiftData)
- **Swift**: 6.0

## Build Commands

```bash
# Generate/regenerate Xcode project (run after any project.yml changes)
xcodegen generate

# Open in Xcode
open Mirari.xcodeproj

# Lint code
swiftlint

# Run tests (from command line)
xcodebuild test \
  -project Mirari.xcodeproj \
  -scheme Mirari \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  CODE_SIGNING_ALLOWED=NO

# Build only (no tests)
xcodebuild build \
  -project Mirari.xcodeproj \
  -scheme Mirari \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  CODE_SIGNING_ALLOWED=NO
```

## CI Pipeline

GitHub Actions runs on PRs and pushes to main:
- **SwiftLint**: Code style checking
- **Build & Test**: Compiles and runs unit tests on iOS Simulator

## Architecture

### Key Components

- `Features/Scanner/` - Camera UI, card detection, and detected card display
- `Features/Collection/` - Collection grid view
- `Services/Gemini/GeminiService.swift` - Gemini AI card detection (frame-by-frame)
- `Services/Scryfall/` - Scryfall API client with rate limiting and 4-strategy fallback lookup

### Key Design Decisions

1. **Scryfall as source of truth**: AI identifies cards, Scryfall provides authoritative data (prices, images, oracle text)
2. **Fallback lookup strategy**: Set+number → name+set → exact name → fuzzy name (see `ScryfallService.lookupFromDetection`)
3. **Confidence scoring**: AI returns confidence level (0.0-1.0) for detection quality
4. **Offline-first collection**: SwiftData stores full card data, not just Scryfall IDs

### Testing

Tests use `MockURLProtocol` to mock network responses without hitting real APIs. See `MirariTests/ScryfallServiceTests.swift` for examples of mocking Scryfall responses.

## Firebase Setup

This app uses Firebase AI Logic for Gemini API access. Firebase must be configured:

1. Create a Firebase project at [Firebase Console](https://console.firebase.google.com/)
2. Enable **AI Logic** and select **Gemini Developer API**
3. Register iOS app with bundle ID `com.mirari.app`
4. Download `GoogleService-Info.plist` and place in `Mirari/` folder
5. Run `xcodegen generate` to include it in the project

The `GoogleService-Info.plist` file is gitignored for security.

## Testing on Device

Camera features require a physical iPhone. Simulator won't work for scanning.

1. Connect iPhone via USB
2. In Xcode: Select your iPhone from device dropdown
3. First time: Trust the developer certificate on iPhone (Settings > General > Device Management)
4. Build and run (Cmd+R)

---

## Implementation Roadmap

- [x] **Phase 1**: Project setup & camera preview
- [x] **Phase 2**: Gemini Frame-by-Frame detection (gemini-3-flash-preview)
- [x] **Phase 3**: Scryfall API integration
- [ ] **Phase 4**: Collection management (CardDetailView)
- [ ] **Phase 5**: CSV import/export
- [ ] **Phase 6**: Gemini Live API streaming (Gemini 2.5 Flash)

---

## Phase 4: Collection Management

**Goal**: View and edit saved cards in collection

**Already implemented**:
- `Card.swift` - SwiftData model with all fields
- `CollectionView.swift` - Grid display of saved cards
- `DetectedCardView.swift` - Save to collection functionality

**Remaining work**:
- `CardDetailView.swift` - View/edit individual cards (quantity, condition, delete)

---

## Phase 5: CSV Import/Export

**Goal**: Export collection to CSV, import from other apps (Deckbox, Moxfield, etc.)

**CSV Format**:
```csv
name,set_code,collector_number,quantity,condition,foil
"Lightning Bolt","2xm","117",4,"NM",false
```

**Files to create**:
- `Mirari/Services/CSVService.swift`
- Wire up Import/Export buttons in `CollectionView.swift`

---

## Phase 6: Gemini Live API (Streaming)

**Goal**: Add real-time streaming detection as alternative scanning mode

**Pre-requisite Refactor**: Extract protocol from GeminiService:
1. Create `CardDetectionProtocol` with `func detectCard(from image: UIImage) async throws -> DetectionResult`
2. Have `GeminiService` conform to it
3. Create `GeminiLiveService` implementing WebSocket streaming
4. Add mode toggle in `ScannerView`

**Live API Notes**:
- Uses `gemini-2.5-flash-preview`
- WebSocket connection for bidirectional streaming
- Send frames as base64-encoded JPEG at ~1 FPS

**Files to create**:
- `Mirari/Services/Gemini/CardDetectionProtocol.swift`
- `Mirari/Services/Gemini/GeminiLiveService.swift`

---

## Future Features (Stretch)

- Similar cards using AI (based on mechanics, art style, color)
- Card generation with Imagen 3
- iCloud sync
- Deck building
- Price tracking

---
> Source: [evansenter/mirari](https://github.com/evansenter/mirari) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
