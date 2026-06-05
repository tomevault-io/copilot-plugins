## outspire

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Outspire is an iOS/macOS campus companion app for WFLA (World Foreign Language Academy) students, built with SwiftUI. It connects to TSIMS (a school information management system) to provide class schedules, scores, CAS (Creativity, Activity, Service) tracking, and school announcements.

- **Bundle ID:** `dev.wrye.Outspire`
- **App Group:** `group.dev.wrye.Outspire`
- **Deployment target:** iOS 17.0 (main app), iOS 18.2 (widget extension)
- **Swift version:** 5.0
- **Scheme:** `Outspire`

## Build & Test Commands

```bash
# Build for simulator
xcodebuild -project Outspire.xcodeproj -scheme Outspire -destination 'platform=iOS Simulator,name=iPhone 17 Pro' build

# Run all tests
xcodebuild -project Outspire.xcodeproj -scheme Outspire -destination 'platform=iOS Simulator,name=iPhone 17 Pro' -enableCodeCoverage YES test

# Lint
swiftlint
swiftformat --lint .

# Auto-format
swiftformat .
```

## Linting Rules

**SwiftLint** (`.swiftlint.yml`): Line length warning at 120, error at 140. Function body max 200/300. File length max 1200/2000. `force_cast` is an error. Identifier names: 2-40 chars.

**SwiftFormat** (`.swiftformat`): 4-space indent, `--self remove`, `--maxwidth 120`, `--wraparguments before-first`. `redundantSelf` and `sortedImports` are disabled.

## Architecture

### Authentication

TSIMS v2 cookie-based auth via `AuthServiceV2` singleton. Uses `TSIMSClientV2` for requests to `Configuration.tsimsV2BaseURL`. Stores credentials in Keychain via `SecureStore` and auto-reauthenticates when sessions expire (~30min). Keeps sessions alive with a 20-minute timer. Optimistic auth on cold launch: cached credentials = immediate `isAuthenticated = true` before network verify.

### TSIMS v2 Service Layer

Domain services under `Core/Services/TSIMS/`:
- `AuthServiceV2` -- Login, logout, session keep-alive, auto-reauth
- `TimetableServiceV2` -- Class schedules (fetches timetable + HTML-scraped year options)
- `ScoreServiceV2` -- Academic scores
- `CASServiceV2` -- Club activities, reflections, group management
- `HomeServiceV2` -- Menu structure

All use `TSIMSClientV2` which wraps `URLSession` with cookie handling, automatic retry on 302/401/HTML (triggers `AuthServiceV2.refreshSessionIfNeeded`), and `ApiResponse<T>` envelope decoding.

### Feature Module Layout

Each feature in `Features/` follows MVVM with `Views/` and `ViewModels/` subdirectories. Main tabs:
- **Today** (`Features/Main/Views/TodayView`) -- Dashboard with class countdowns, gradient system, quick links
- **Class** (`Features/Academic`) -- Timetable viewer, biometric-protected score viewer
- **Activities** (`Features/CAS`) -- Club info, activity records, reflections with LLM suggestions
- **Explore** (`Features/SchoolArrangement`, `Features/Map`, `Features/Settings`) -- Weekly arrangements, lunch menus, campus map, settings

### Navigation

`RootTabView` provides three tab implementations branching on iOS version: iOS 26+ (Liquid Glass with `.sidebarAdaptable`), iOS 18+, and legacy. `NavSplitView` provides iPad split view. Deep linking via `URLSchemeHandler` supports `outspire://` URL schemes and universal links (`outspire.wrye.dev`).

### Widget & Live Activities

`OutspireWidget/` is a separate target sharing data via the `group.dev.wrye.Outspire` app group. `WidgetDataManager` writes to shared `UserDefaults`; `WidgetDataReader` reads in widget process. `ClassActivityManager` manages Live Activity lifecycle with push-driven updates via Cloudflare Worker (`outspire-apns.wrye.dev`). `PushRegistrationService` handles device registration with SHA256 deduplication.

### Design System

`UI/Theme/DesignTokens.swift` defines spacing (`AppSpace`), radius (`AppRadius`), shadow (`AppShadow`), and color (`AppColor`) tokens. Brand tint from asset catalog (`BrandTint`). Rich dark mode surfaces (blue-black tones, not flat gray). `GlassmorphicComponents` provides card modifiers with iOS 26+ Liquid Glass fallbacks. `GradientManager` drives context-aware dynamic gradients (subject-specific colors during active classes).

### Caching

`CacheManager` provides UserDefaults-based caching with TTL. Key durations: timetable 1 day, scores/terms/activities 5 minutes, arrangements 24 hours. Daily automatic cleanup of expired entries. ViewModels serve stale cache during network failures.

### Logging

Uses OSLog via `Log` enum (`Log.app`, `Log.net`, `Log.auth`, `Log.widget`) with subsystem `dev.wrye.Outspire`.

### Configuration

- `Configurations.swift` -- Feature flags and user preferences, backed by `UserDefaults`. Server URLs, holiday mode, debug toggles, LLM model.
- `Configurations.local.swift` -- Git-ignored file for secrets (LLM API key, push worker auth). Copy from `Configurations.local.swift.example`.

## SPM Dependencies

- **SwiftSoup** -- HTML parsing (TSIMS v2 profile/timetable scraping)
- **Toasts** (swiftui-toasts) -- Toast notifications
- **ColorfulX** -- Animated gradient backgrounds
- **SwiftOpenAI** -- LLM API client for CAS suggestions (Grok-3)

## Documentation

Full project wiki lives in `docs/wiki/`. Start at [docs/wiki/Home.md](docs/wiki/Home.md).

**When making changes to the code structure, architecture, or adding/removing features, you must update the relevant wiki pages and add an entry to `docs/CHANGELOG.md`.** This includes:
- Adding, renaming, or removing source files or directories
- Changing service APIs, model structures, or data flow
- Adding or removing dependencies
- Modifying navigation, deep linking, or tab structure
- Changes to the design system, caching strategy, or auth flow

## Commit Messages

Never add Co-Authored-By lines for Claude/AI in commit messages.

---
> Source: [Computerization/Outspire](https://github.com/Computerization/Outspire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
