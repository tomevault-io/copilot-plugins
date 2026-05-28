## whatport

> macOS menu bar utility showing real-time USB-C port status (protocol, speed, power).

# WhatPort

macOS menu bar utility showing real-time USB-C port status (protocol, speed, power).

## Stack

- Swift 6 (strict concurrency)
- SwiftUI (menu bar UI, charts)
- IOKit C APIs (wrapped, never exposed above the IOKit layer)
- Minimum deployment: macOS 14.0 (Sonoma)
- Apple Silicon only (M1+)

## Architecture

Three layers. Each depends only on the one below it. Never skip a layer.

```
SwiftUI (WhatPortUI)  ->  Domain (WhatPortCore)  ->  IOKit (WhatPortIOKit)
```

Rules:
- IOKit layer: owns all C API interaction. Nothing above touches `io_service_t`, `IORegistryEntry`, `CFMutableDictionary`, or any C pointer type.
- Domain layer: pure Swift. No IOKit imports, no AppKit/SwiftUI imports. All business logic lives here.
- SwiftUI layer: rendering and formatting only. No data fetching, no business logic.
- Formatting (e.g. "5.9W", "40 Gbps x 2") happens in the SwiftUI layer, not the domain model.

## Conventions

- Named exports preferred
- Small focused functions that do one thing
- Early returns to reduce nesting
- Descriptive variable names, no abbreviations
- Handle errors explicitly, never swallow silently
- No over-engineering. Solve the problem at hand.
- Match existing patterns before introducing new ones
- Comments only when the WHY is non-obvious

## IOKit specifics

- Match PHY services via superclass name `AppleTypeCPhy` (not the chip-specific `AppleT8132TypeCPhy`)
- Socket ID on IOThunderboltPort = physical port number (string, parse to Int)
- PowerOutDetails must be polled (3s timer). No notification fires for power updates.
- Connection events use IOKit interest notifications (kIOGeneralInterest on IOPortTransportStateCC)
- All IOKit reads are unprivileged. No root, no entitlements needed.

## Build and run

```bash
# Build from command line
xcodebuild -scheme WhatPort -configuration Debug build

# Run
open build/Debug/WhatPort.app
```

## Testing

- Domain layer is testable in isolation (no IOKit dependency, protocol-based data source)
- Use protocol conformances to mock IOKit data in tests
- Test with real hardware where possible (plug/unplug scenarios)

## Key references

- Spec: SPEC.md in this directory
- Research data: ../whatcable-app/research/ (probes, data source docs, field references)
- Bundle ID: uk.whatport.whatport

---
> Source: [darrylmorley/whatport](https://github.com/darrylmorley/whatport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
