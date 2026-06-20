## everythingclient

> When building for testing, use the Xcode build script instead of `swift build`, replace `EverythingClient` with the target I'm currently working on:

# Cursor Rules for everythingclient project

## Build Commands
When building for testing, use the Xcode build script instead of `swift build`, replace `EverythingClient` with the target I'm currently working on:
```bash
scheme=EverythingClient bash .github/scripts/build.sh
```

## Project Structure
- This is an iOS project using Xcode workspace, feature based modularization
- Each module has a different architecture (such as MVVM, Clean architecture, ...)
- Main scheme: EverythingClient

## Development Guidelines
- when working on a module, look at the structure of the module to see if whether it is mainly SwiftUI or RxSwift or UIKit, and what the architecture is (MVVM, Clean, ...) before start writing code
- look at root level **Package.swift** first to know the dependencies before start implementation

---
> Source: [mangosteencorp/everythingclient](https://github.com/mangosteencorp/everythingclient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
