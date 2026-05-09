## tinydependency

> TinyDependency is a small Swift Package that provides lightweight dependency injection for non-view code in SwiftUI-oriented projects.

# AGENTS.md

## Project Summary

TinyDependency is a small Swift Package that provides lightweight dependency injection for non-view code in SwiftUI-oriented projects.

Core source files:

- `Sources/TinyDependency/DependencyKey.swift`
- `Sources/TinyDependency/DependencyValues.swift`
- `Sources/TinyDependency/Dependency.swift`
- `Sources/TinyDependency/WithDependencies.swift`
- `Sources/TinyDependency/Internal/Storage.swift`

## Build And Test

Primary commands:

- `swift build`
- `swift test`
- `swift run DependencyContextProbe`

Use `swift test` for the normal suite.
Use `swift run DependencyContextProbe` when a scenario must be verified in a real non-test process.
The automated test suite also launches `DependencyContextProbe` as a subprocess for context integration checks.

## Dependency Context Rules

TinyDependency resolves default dependency values in this order:

1. `TINY_DEPENDENCY_CONTEXT`
2. Xcode Preview detection
3. Real test-environment detection
4. Default app runtime

Supported `TINY_DEPENDENCY_CONTEXT` values:

- `default`
- `live` (alias of `default`)
- `test`
- `preview`

If `TINY_DEPENDENCY_CONTEXT=test` is set, `testValue` must be selected even in a normal non-test process.
Unknown override values should be ignored and must fall back to automatic detection.

## Non-Test Probe

`DependencyContextProbe` exists specifically for context-selection checks that should not rely on the test runner.
It is both a manual probe and a subprocess fixture used by `Tests/TinyDependencyTests/TinyDependencyTests.swift`.

Expected output:

- no override: `ProductionProbeLogger`
- `TINY_DEPENDENCY_CONTEXT=test`: `TestProbeLogger`
- `TINY_DEPENDENCY_CONTEXT=preview`: `PreviewProbeLogger`
- `TINY_DEPENDENCY_CONTEXT=live`: `ProductionProbeLogger`

Recommended commands:

```bash
swift run DependencyContextProbe
env TINY_DEPENDENCY_CONTEXT=test swift run DependencyContextProbe
env TINY_DEPENDENCY_CONTEXT=preview swift run DependencyContextProbe
env TINY_DEPENDENCY_CONTEXT=live swift run DependencyContextProbe
```

Do not use the normal test process to validate those non-test expectations.

## Documentation Rules

When changing context resolution, platform requirements, or installation instructions, update both:

- `README.md`
- `README_CN.md`

If the package license changes, keep `LICENSE` aligned with the README badge/link.

## Release Notes

This repository uses semantic version tags in the form `vX.Y.Z`.

When preparing a release, confirm:

- tests pass with `swift test`
- context behavior matches the probe output
- README and README_CN stay in sync

---
> Source: [fatbobman/TinyDependency](https://github.com/fatbobman/TinyDependency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
