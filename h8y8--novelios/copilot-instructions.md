## novelios

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**NovelNarrator** — iOS app for importing `.txt` novels, reading them in an immersive reader, and narrating them via TTS (Text-to-Speech) with background playback support.

- Language: Swift 5.9+, iOS 17+ minimum
- UI: SwiftUI only (no UIKit unless wrapping is required)
- Persistence: SwiftData (`Book`, `Chapter`, `UserSettings` models)
- TTS: `AVSpeechSynthesizer` with `zh-TW` locale, `AVAudioSession` `.playback` category for background audio
- Lock screen integration: `MPNowPlayingInfoCenter` + `MPRemoteCommandCenter`
- Concurrency: Swift Concurrency (`async/await`), no Combine unless necessary

## Build & Run

Open `novel/novel.xcodeproj` in Xcode 15+ and run on a simulator or device. There is no package manager; all dependencies are system frameworks.

To run tests:
- Unit tests: `novelTests` target (`novel/novelTests/novelTests.swift`)
- UI tests: `novelUITests` target

From command line:
```bash
xcodebuild -project novel/novel.xcodeproj -scheme novel -destination 'platform=iOS Simulator,name=iPhone 16' build
xcodebuild -project novel/novel.xcodeproj -scheme novelTests -destination 'platform=iOS Simulator,name=iPhone 16' test
```

## Architecture

### Data Flow
`LibraryView` → user taps a `BookCard` → `ReaderView` (receives `@Bindable Book`) → reads chapter content via `book.chapterContent(at:)` using UTF-16 offsets → passes paragraphs to `ScrollReaderView` + `TTSService`

### Key Services (all in `novel/novel/Services/`)
- **`TTSService`** — `@Observable` class injected via `.environment(ttsService)` at app root. Manages `AVSpeechSynthesizer` state, paragraph-by-paragraph narration, and exposes `onChapterFinished` callback for auto-advance.
- **`NowPlayingService`** — Manages `MPNowPlayingInfoCenter` and `MPRemoteCommandCenter` for lock screen controls. Created as `@State` inside `ReaderView`.
- **`ChapterParser`** — Static struct. Parses chapter boundaries via regex (`第X章/回/節`, `Chapter X`, `卷X`). Falls back to 3000-char chunks if no patterns match.
- **`EncodingDetector`** — Detects UTF-8 / Big5 / GBK encoding on import.

### Models (SwiftData, `novel/novel/Models/`)
- `Book` — stores full `content: String`, has cascade-delete `chapters: [Chapter]`. Chapter content is accessed via UTF-16 offset slicing (`chapterContent(at:)`).
- `Chapter` — `startOffset`/`endOffset` are **UTF-16 offsets** into `Book.content`.
- `UserSettings` — singleton pattern: `@Query` fetches all, uses `.first ?? UserSettings()`. Has computed `readingTheme: ReadingTheme` bridging to the `ReadingTheme` enum.

### Views (SwiftUI)
- `ReaderView` — orchestrates toolbars, TTS controls, chapter navigation, progress saving, and NowPlaying setup. Wraps `ScrollReaderView` for content display.
- `ScrollReaderView` — renders paragraphs with highlight support for TTS current paragraph.
- `SettingsSheet`, `ChapterListSheet` — presented as `.sheet` from `ReaderView`.

### Theme
`ReadingTheme` enum (`novel/novel/Theme/ReadingTheme.swift`) — four themes: `.light`, `.sepia`, `.gray`, `.dark`. Provides `backgroundColor`, `textColor`, `highlightColor`, `toolbarStyle` (Material).

## Important Conventions

- `TTSService` is instantiated once in `novelApp` and passed as environment object — never instantiate it inside views.
- Chapter offsets use **UTF-16 units** throughout — be careful when slicing `String` content.
- `UserSettings` is a SwiftData singleton: always use `allSettings.first ?? UserSettings()` pattern; insert a new one if `allSettings.isEmpty`.
- All user-visible Chinese strings should use `LocalizedStringKey` to preserve future localization support.
- Background audio requires `UIBackgroundModes: audio` in `Info.plist` and `AVAudioSession` category `.playback` set at app launch (`novelApp.init()`).
- `AVSpeechSynthesizerDelegate` callbacks come off the main actor — use `Task { @MainActor in ... }` to update `@Observable` state.

---
> Source: [H8Y8/noveliOS](https://github.com/H8Y8/noveliOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
