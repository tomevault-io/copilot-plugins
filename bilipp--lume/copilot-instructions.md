## lume

> Lume is a native, multi-platform IPTV player (iOS 18+, macOS 15+, tvOS 18+, visionOS 2+) built with SwiftUI + SwiftData. Single Swift codebase with three interchangeable playback engines: KSPlayer (default) → VLCKit → AVPlayer. It is built with the iOS 26 SDK and uses Liquid Glass / iOS 26 navigation APIs where available, falling back to system materials on older OS versions.

# Lume — AI Agent Guide

Lume is a native, multi-platform IPTV player (iOS 18+, macOS 15+, tvOS 18+, visionOS 2+) built with SwiftUI + SwiftData. Single Swift codebase with three interchangeable playback engines: KSPlayer (default) → VLCKit → AVPlayer. It is built with the iOS 26 SDK and uses Liquid Glass / iOS 26 navigation APIs where available, falling back to system materials on older OS versions.

---

## Build & run

```bash
# Open in Xcode
open Lume.xcodeproj   # pick scheme "Lume", any destination

# CLI build (iOS Simulator)
xcodebuild build \
  -project Lume.xcodeproj -scheme Lume \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro'
```

The project injects API secrets from a repo-root `.env` file via `Scripts/inject-env.sh`. The file is gitignored; features degrade gracefully when it's absent.

---

## Testing

Tests deploy to **iOS 26.4+ Simulator only** — never tvOS. Use an iPhone 17 Pro or newer sim; iOS 26.2 sims fail with a deployment-target mismatch (exit 65).

```bash
# Full suite
xcodebuild test -project Lume.xcodeproj -scheme Lume \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro'

# Unit tests only
xcodebuild test -project Lume.xcodeproj -scheme Lume \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro' \
  -only-testing:LumeTests

# UI tests only
xcodebuild test -project Lume.xcodeproj -scheme Lume \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro' \
  -only-testing:LumeUITests
```

Every test `ModelConfiguration` must set `cloudKitDatabase: .none` — `@Attribute(.unique)` models + the default `.automatic` crashes on entitled simulator hosts.

---

## Architecture

```
Lume/
├── LumeApp.swift            App entry + SwiftData containers
├── Models/                  SwiftData @Model types (Playlist, LiveStream, Movie, Series, …)
├── Services/
│   ├── Network/             XtreamClient, M3UClient, TMDBClient, OMDbClient, TraktService
│   ├── Sync/                ContentSyncManager (background catalog indexing + enrichment)
│   ├── Player/              PlayerSettings, PlayerHistory, NextUp resolver
│   └── Images/              CachedAsyncImage, ImagePipeline
└── Views/                   SwiftUI, platform-adaptive
    ├── Home/                Hero carousel, rails, tvOS fold
    ├── Player/              Engine wrappers + unified overlay
    └── …
```

Two separate `ModelContainer`s:
- **Catalog** (`default.store`) — local-only, what all `@Query` bindings target
- **CloudKit mirror** (`CloudUserData.store`) — user state (profiles, watch progress, favorites); never bind `@Query` against this container

---

## Key patterns & gotchas

### Swift concurrency
`SWIFT_DEFAULT_ACTOR_ISOLATION=MainActor` is set project-wide. Value types and DTOs used by `nonisolated` callers must be explicitly marked `nonisolated` (type + every extension).

### SwiftData
- Enrichment saves run on a **background** `ModelContext` (`ContentSyncManager.enrich*`) — not the view context.
- Main-thread saves during playback stall KSPlayer every ~5 s. Buffer to `UserDefaults`; flush at playback boundaries.
- `PlaylistDeletion` helper must be used for any playlist removal (UI **and** iCloud reconcile) — `Movie`/`Series`/`LiveStream` have no cascade relationship to `Playlist`.
- The reconciler after `switchProfile` rebuilds the dropped content shadow — don't remove that pass; optimize the fetch predicate instead.

### iCloud sync
- Guard reconcile against `LocalCatalogReadiness`; an empty `default.store` would push mass deletions to CloudKit.
- `UserProfile` must be deduped on every reconcile (not just launch) — fixed-id default profiles multiply per device in CloudKit.

### tvOS-specific
- `Color.accentColor` resolves to white on tvOS — never use it for fills/tints.
- `.onMoveCommand` runs inside the focus engine's animated context; defer layout mutations with `Task { }`.
- Full-width focus targets needed for vertical navigation — a narrow target won't catch "down" from a full-width section.
- `@FocusState` must not drive layout sizing in the hero fold — use `TVHomeScreen`'s `ScrollTargetBehavior`.

### KSPlayer
- Hardware decode requires **both** `asynchronousDecompression = true` **and** `hardwareDecode = true`; `async` defaults to `false` → silent software decode → frame drops on tvOS.
- Never call `layer.prepareToPlay()` on a running session — use `player.replace()` (`rebuildStream(on:)`) to avoid a UAF crash.
- Frozen image + healthy audio on live TV = MPEG-TS 2³³ clock wrap; fixed by the `noteClockDrift()` watchdog.

### Localization
String Catalogs (English + German). Run `xcstringstool sync` and include the tvOS stringsdata. Normalize `.xcstrings` with `Scripts/normalize-xcstrings.swift` (pre-commit hook) to avoid format churn.

### Pre-commit hooks (lefthook)
SwiftFormat + SwiftLint run as errors. Notable: `String(decoding:)` is banned; `redundantStaticSelf` crashes on `for x in (try? …) ?? []` — avoid that pattern.

---

## External integrations

| Service | Auth | Notes |
|---------|------|-------|
| TMDB | Bearer token (`.env`) | Metadata, artwork, trailers |
| OMDb | API key (`.env`) | IMDb / RT / Metacritic ratings |
| Trakt | Device OAuth (Keychain) | Scrobbling; no web view — works on tvOS |

---

## GitHub
Issues & roadmap: <https://github.com/bilipp/Lume/issues>

---
> Source: [bilipp/Lume](https://github.com/bilipp/Lume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
