## teleport

> - Teleport is a native macOS SwiftUI app for simulating iOS device location on simulators and physical devices over USB or Wi-Fi.

# Project Guidelines

## Project Nature

- Teleport is a native macOS SwiftUI app for simulating iOS device location on simulators and physical devices over USB or Wi-Fi.
- Keep this file preference-oriented and concise; use the README for broader project details.

## Build And Validation

- Prefer Makefile targets whenever an equivalent exists instead of invoking the underlying tool directly, for example `make format`, `make lint`, `make build`, or `make launch`.
- Before commit, run `make format`.
- When creating new files, make sure that Xcode includes them in the target membership for the `Teleport` app target. Otherwise the build will fail.

---
> Source: [samuelhe52/Teleport](https://github.com/samuelhe52/Teleport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
