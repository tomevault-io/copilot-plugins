## app-launcher

> - Build the Swift package with `swift build`.

# AGENTS.md

## Build And Run

- Build the Swift package with `swift build`.
- Run the app locally with `swift run AppLauncher`.
- The package targets macOS 26.5 or newer and Swift 6.3 or newer.

## Test

- Run the full test suite with `swift test`.
- Add or update tests for every behavior change.
- Keep tests focused on the changed behavior and prefer existing test patterns in `Tests/AppLauncherTests`.

## Rules

- Always add or update `REQUIREMENTS.md` when product behavior, platform support, user-facing flows, or constraints change.
- Always add or update tests when implementation behavior changes.
- Always build the Swift package with `swift build` after changes are done.
- Always rebuild `App Launcher.app` with `scripts/setup-app-bundle.sh` after changes are done.
- Keep changes scoped to the requested work and follow the existing Swift and SwiftUI style.
- Prefer Swift Package Manager commands unless the project adds an Xcode workspace or project.

---
> Source: [lgrammel/app-launcher](https://github.com/lgrammel/app-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
