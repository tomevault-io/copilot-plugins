## twinkle

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
# Build the package
swift build

# Run all tests
swift test

# Run a single test suite
swift test --filter TwinkleTests.ReleaseClientTests

# Run a single test
swift test --filter "ReleaseClientTests/previewFetchReleases"
```

## Architecture Overview

Twinkle is a macOS app auto-updater library that checks GitHub releases for updates. It uses [Point-Free's Dependencies](https://github.com/pointfreeco/swift-dependencies) for dependency injection and [swift-sharing](https://github.com/pointfreeco/swift-sharing) for persisted state.

### Core Components

**Twinkle** (`Twinkle.swift`) - Main `@Observable` class that orchestrates the update lifecycle:
- Fetches releases from GitHub API
- Compares build numbers to determine if update is available
- Downloads, extracts, validates code signing, and installs updates
- Uses `NSBackgroundActivityScheduler` for periodic background checks

**UpdateState** - Enum representing the update state machine:
`idle → checking → upToDate | available → downloading → ready → installing | failed`

### Dependency Clients

All clients use `@DependencyClient` macro with `liveValue`, `previewValue`, and `testValue` implementations:

- **ReleaseClient** - Fetches releases from GitHub API, downloads zip files with progress
- **BundleInfoClient** - Accesses bundle info (version, identifier, URL)
- **ProcessClient** - Shell commands for `unzip` and `codesign` validation

### Shared State Keys (AppStorage)

Defined in `SharedKeys.swift`:
- `lastUpdateCheck` - Timestamp of last check
- `ignoredVersion` - User-skipped version
- `betaUpdatesEnabled` - Include prereleases

### Release Model

Releases are identified by `version` string and compared by `buildNumber` (Int). The JSON expects fields: `build`, `version`, `changelog`, `bannerImageUrl`, `zipUrl`, `prerelease`.

## Testing Pattern

Tests use Swift Testing framework (`@Test`, `#expect`) with Point-Free's `withDependencies` for injecting mock clients.

---
> Source: [Aayush9029/twinkle](https://github.com/Aayush9029/twinkle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
