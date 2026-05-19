## bandwidth

> This file gives Claude Code project-specific guidance for working on Bandwidth Guard.

# CLAUDE.md

This file gives Claude Code project-specific guidance for working on Bandwidth Guard.

## Project Overview

Bandwidth Guard is a native macOS menu bar app built with Swift 6, SwiftUI, Swift Package Manager, and Tuist. The app is structured as small modules instead of one large app target.

## Important Commands

Run tests:

```bash
swift test
```

Format code:

```bash
swiftformat .
```

Check formatting:

```bash
swiftformat --lint .
```

Run lint checks:

```bash
swiftlint --strict
```

Generate the Xcode workspace:

```bash
tuist generate run --no-open
```

Build the app through Tuist:

```bash
tuist xcodebuild build -workspace BandwidthGuard.xcworkspace -scheme BandwidthGuard -configuration Debug CODE_SIGNING_ALLOWED=NO
```

Build the Network Extension scaffold:

```bash
tuist xcodebuild build -workspace BandwidthGuard.xcworkspace -scheme BandwidthGuardNetworkExtension -configuration Debug CODE_SIGNING_ALLOWED=NO
```

Package a local release artifact:

```bash
./Scripts/package-release.sh v0.1.0
```

## Architecture Rules

- `Modules/Core` owns models, persistence contracts, app state, and business rules
- `Modules/Core` must not import SwiftUI, AppKit, or NetworkExtension
- `Modules/Discovery` may import AppKit and Core, but must not depend on UI
- `Modules/UI` may import SwiftUI and Core, but should not own persistence details
- `Apps/BandwidthGuard` wires modules together and owns app lifecycle concerns
- Add protocols only for real boundaries, test seams, or second implementations

## Editing Guidance

- Prefer small, direct changes over broad refactors
- Keep SwiftFormat and SwiftLint passing before finishing work
- Add or update tests for behavior changes in Core
- Avoid adding new modules until the boundary is proven by real code
- Do not introduce signing, notarization, or entitlement changes unless explicitly requested
- Do not commit generated Xcode workspaces or build artifacts

## Current Release State

Release archives are unsigned and not notarized. Public distribution still needs Developer ID signing, hardened runtime validation, notarization, and final release packaging.

---
> Source: [alhasnalshrif/bandwidth](https://github.com/alhasnalshrif/bandwidth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
