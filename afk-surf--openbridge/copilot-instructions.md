## openbridge

> This file provides guidance to LLM coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to LLM coding agents when working with code in this repository.

> **Note**: `CLAUDE.md` is a symlink to this file.

## Project Overview

OpenBridge is a macOS-first local agent chat app. It preserves the native macOS chat experience, embedded WebViews, skills, schedules, memory, computer-use workflows, and sandbox VM execution while removing login, payment, usage, backend-hosted LLM clients, clipboard history, and Lens/screenshot capture features.

The local agent runtime is built around vendored `kwwk` in this repository. Keep `kwwk` broadly useful as an agent SDK; avoid adding OpenBridge-specific business concepts to it unless they are general SDK primitives.

## Repository Structure

```
openbridge/
├── sandbox-vm/          # Go sandbox VM control library and macOS runtime framework
├── macos/               # SwiftUI macOS application
├── kwwk/                # Vendored local agent SDK/runtime
├── web/                 # Embedded WebView React assets
│   └── src/
│       ├── assets/      # Shared assets
│       ├── embedded/    # WebViews embedded in macOS app
│       └── utils/       # Shared utilities and JSBridge client
├── scripts/             # Shared utility scripts
├── docs/                # Project documentation
└── .github/workflows/   # CI pipelines
```

## Component Guidelines

- **[sandbox-vm/AGENTS.md](sandbox-vm/AGENTS.md)** — Go sandbox VM control library and macOS runtime framework
- **[macos/AGENTS.md](macos/AGENTS.md)** — SwiftUI macOS application
- **[web/AGENTS.md](web/AGENTS.md)** — TypeScript/React embedded WebViews
- **[macos/DevKit/XcodeStringsHelper/AGENTS.md](macos/DevKit/XcodeStringsHelper/AGENTS.md)** — Xcode `.xcstrings` localization tooling

## Agent Guide

- Do not generate markdown documents after files changed; output directly.
- Do not add comments solely for chat information; comments must have long-term value.
- Limit batch sizes to about 100 items when generating large outputs.
- Clean up temporary files after completing generation tasks.

## Common Principles

- Use early returns to reduce nesting.
- Keep functions and methods focused and small.
- Use meaningful variable names.
- Maintain one authoritative source for each piece of logic.
- Prefer dependency injection over singletons.
- Prefer composition over inheritance.

## Build Commands

Prefer the provided scripts and `make` targets for day-to-day work. CI, Makefiles, and repo scripts may invoke underlying tools directly; avoid running low-level build commands manually unless debugging the scripts themselves.

| Component | Build Command | Test Command |
|-----------|--------------|--------------|
| macOS App | `cd macos && BUILD_CONFIGURATION=UnsignedDebug bash DevKit/Scripts/workspace_build_debug.sh` | `cd macos && bash DevKit/Scripts/workspace_test_unit.sh` |
| Sandbox VM | `make -C sandbox-vm framework` | `make -C sandbox-vm go-test` |
| Web embedded assets | `cd web && yarn build:embedded` | — |
| kwwk | `cd kwwk && swift build` | `cd kwwk && swift test` |

## Fresh Worktree Bootstrap

On a fresh worktree, build embedded WebView resources before building the macOS app:

```bash
cd web && yarn install --immutable && yarn build:embedded
cd macos && BUILD_CONFIGURATION=UnsignedDebug bash DevKit/Scripts/workspace_build_debug.sh
```

Before launching a freshly built app, close any already-running `OpenBridge` instances first:

```bash
pkill -f '/OpenBridge.app/Contents/MacOS/OpenBridge' || true
open -na "/absolute/path/to/macos/.build/DerivedData/Build/Products/UnsignedDebug/OpenBridge.app"
```

When iterating on `web/src/embedded/chat` with `cd web && yarn serve:chat`, do not rebuild or relaunch the macOS app for every chat-only change. Rebuild/relaunch only when native macOS code, JS bridge/native integration, or packaged `WebKitBridgeResources` change.

## Language-Specific Guidelines

### Go

- Go 1.24+ required.
- Prefer `make` targets for local work.
- Tabs for indentation.
- Wrap errors with context: `fmt.Errorf("context: %w", err)`.
- Use `context.Context` for cancellation and timeouts.
- Define interfaces where used, not where implemented.
- Use table-driven tests with `_test.go` suffix.

### Swift

- 4 spaces for indentation.
- Prefer DevKit build scripts; avoid invoking `xcodebuild` or `swift build` manually for the app.
- Use `@Observable` macro rather than `ObservableObject`/`@Published`.
- Use Swift concurrency: `async/await`, `Task`, `actor`.
- Split files using the `Xxx+Xxx.swift` pattern.
- Use `String(localized:)` for user-facing strings.
- Never edit `.xcstrings` directly; use `i18n.py apply`.

### TypeScript/React

- Use yarn for package management and rspack for bundling.
- Use oxlint and prettier for code quality.
- Use Tailwind CSS for styling.
- Keep `web/src/embedded` focused on WebViews embedded in the macOS app.

---
> Source: [AFK-surf/OpenBridge](https://github.com/AFK-surf/OpenBridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
