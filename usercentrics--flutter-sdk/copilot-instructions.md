## flutter-sdk

> Swift coding conventions for the iOS plugin


# Swift Conventions (iOS Plugin)

## Project Setup

- Swift 5.3, iOS deployment target: 11.0
- Pod: `usercentrics_sdk`, depends on `UsercentricsUI`
- Plugin registration via ObjC (`UsercentricsPlugin.m/.h`)

## Architecture

- `SwiftUsercentricsPlugin.swift` - Main plugin logic
- `API/` - Proxy abstractions (`UsercentricsProxy`, `UsercentricsBannerProxy`, `FlutterAssetProvider`)
- `Bridge/` - One bridge per SDK method, conforming to `MethodBridge`
- `Serializer/` - One serializer per data type

## Patterns

- Each bridge class handles one MethodChannel call
- Serializers convert native Usercentrics types to Flutter-compatible dictionaries
- Tests live in `example/ios/RunnerTests/` using XCTest
- `UCFlutterFlag` controls feature flags

## Code Style

- Follow Swift API design guidelines
- Use `struct` for value types, `class` for reference types
- Keep bridge classes single-responsibility
- Prefer guard-let over nested if-let

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Usercentrics)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Usercentrics)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
