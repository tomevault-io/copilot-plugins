## musaverakit

> MusaveraKit is a small Swift package that adds app-friendly conveniences around

# MusaveraKit Maintainer Guide

## Purpose

MusaveraKit is a small Swift package that adds app-friendly conveniences around
Apple's MusicUnderstanding framework. Keep the public surface focused on common
music-analysis workflows while preserving direct access to Apple's result
types.

## Requirements

- Xcode 27 beta or later
- Swift 6.4 or later
- A macOS 27 runtime for executing package tests

Set `DEVELOPER_DIR` to the local Xcode 27 installation when it is not selected
globally.

## Repository Layout

- `Sources/MusaveraKit/Analysis`: asynchronous analysis entry points
- `Sources/MusaveraKit/Models`: package-owned result, option, and error types
- `Sources/MusaveraKit/Extensions`: focused conveniences on Apple result types
- `Sources/MusaveraKit/Documentation.docc`: package documentation and guides
- `Tests/MusaveraKitTests`: Swift Testing suites
- `Scripts`: repository maintenance and validation commands

## Design Rules

- Keep `Musavera` as the primary facade for starting analysis.
- Return MusicUnderstanding result types from focused APIs when a wrapper adds
  no meaningful value.
- Add package-owned types only when they simplify several framework types or
  establish a stable package contract.
- Preserve Swift strict-concurrency correctness and `Sendable` boundaries.
- Avoid third-party runtime dependencies.
- Use MusicUnderstanding terminology in public names and documentation.
- Prefer small extensions grouped by the Apple type they enhance.
- Do not commit copyrighted audio, Apple Music credentials, signing material,
  or user-library data.

## Validation

Run the lightweight repository checks with the selected Xcode:

```bash
Scripts/validate-repository.sh
```

Run the complete package tests with Xcode 27:

```bash
swift test
```

Build an Apple-platform target:

```bash
xcodebuild \
  -scheme MusaveraKit \
  -destination 'generic/platform=iOS' \
  -derivedDataPath .build/xcode \
  build
```

Generate DocC documentation after changing public APIs:

```bash
swift package generate-documentation --target MusaveraKit
```

## Pull Requests

- Keep changes focused and split unrelated concerns into separate commits.
- Add tests for package-owned behavior.
- Document public APIs with DocC comments.
- State the exact Xcode and operating-system builds used for runtime checks.
- Call out behavior inherited from a beta Apple SDK instead of hiding it behind
  a package workaround.

See `CONTRIBUTING.md` for the contributor-facing workflow and `RELEASING.md`
for the release checklist.

---
> Source: [rryam/MusaveraKit](https://github.com/rryam/MusaveraKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
