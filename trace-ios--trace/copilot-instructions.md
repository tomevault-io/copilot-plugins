## trace

> `Trace.xcodeproj` is the canonical entry point for the app, packet-tunnel extension, and widget. Core source lives in feature-oriented Swift modules: `Trace/` for the app shell, `TraceCore/` for shared models/storage/logging, `TraceFeatures/` for product features, `TraceUI/` for reusable SwiftUI components, and `TraceVPN/`, `TraceProxy/`, `TraceQUIC/`, `TraceSecurity/`, and `TraceNetworking/` for networking infrastructure. Tests are split across `TraceTests/`, `TraceLogicTests/`, and `TraceUITests/`. Keep generated QUIC artifacts under `build/quiche/` and vendor code under `ThirdParty/` untouched unless the dependency itself is being updated.

# Repository Guidelines

## Project Structure & Module Organization
`Trace.xcodeproj` is the canonical entry point for the app, packet-tunnel extension, and widget. Core source lives in feature-oriented Swift modules: `Trace/` for the app shell, `TraceCore/` for shared models/storage/logging, `TraceFeatures/` for product features, `TraceUI/` for reusable SwiftUI components, and `TraceVPN/`, `TraceProxy/`, `TraceQUIC/`, `TraceSecurity/`, and `TraceNetworking/` for networking infrastructure. Tests are split across `TraceTests/`, `TraceLogicTests/`, and `TraceUITests/`. Keep generated QUIC artifacts under `build/quiche/` and vendor code under `ThirdParty/` untouched unless the dependency itself is being updated.

## Build, Test, and Development Commands
Use Xcode for day-to-day app work:

```bash
open Trace.xcodeproj
xcodebuild -project Trace.xcodeproj -scheme Trace -configuration Debug -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16,OS=latest' build-for-testing
xcodebuild -project Trace.xcodeproj -scheme Trace -configuration Debug -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16,OS=latest' test-without-building
swiftformat --lint . --config .swiftformat
swiftlint lint --strict --config .swiftlint.yml
```

Run `./Tools/QUIC/build_quiche_apple.sh` and `./Tools/QUIC/verify_quiche_symbols.sh build/quiche/quiche.xcframework` only when touching QUIC or `ThirdParty/quiche`.

## Coding Style & Naming Conventions
This repo targets Swift 6. Use 4-space indentation, keep imports sorted, and follow existing module boundaries instead of introducing cross-cutting helpers in the app target. Types use `UpperCamelCase`; methods, properties, and tests use `lowerCamelCase`; test files end in `*Tests.swift`. SwiftFormat keeps explicit `self` in place, and SwiftLint allows longer SwiftUI files, so prioritize clarity over forced decomposition.

## Testing Guidelines
Add or update tests for behavior changes. Put app and integration coverage in `TraceTests/`, lower-level logic regressions in `TraceLogicTests/`, and UI flows in `TraceUITests/`. Prefer focused XCTest names such as `testReplayPreservesHeaders()`. Simulator runs cover most code, but VPN, packet tunnel, and full-tunnel validation still require a physical device.

## Commit & Pull Request Guidelines
Follow Conventional Commits, matching recent history such as `fix(breakpoint): preserve edits` or `refactor(storage): narrow persistence APIs`. Keep PRs small, explain what changed and why, link related issues, and include screenshots or recordings for UI changes. Note signing, entitlement, bundle ID, or migration impacts explicitly.

## Security & Configuration Tips
Do not publish security issues in public tickets; use GitHub Security Advisories instead. When changing signing or bundle IDs, update the app, `TraceVPN`, `TraceWidgetExtension`, entitlements, and `providerBundleIdentifier` in `TraceFeatures/VPN/VPNManager.swift` together.

---
> Source: [Trace-iOS/Trace](https://github.com/Trace-iOS/Trace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
