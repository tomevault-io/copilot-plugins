## clean-shortcuts

> This file is for AI coding assistants (Claude Code, Codex, Cursor, Copilot CLI, etc.) and human contributors working in this repository. `CLAUDE.md` is a symlink to this file.

# Agent / Contributor Guide

This file is for AI coding assistants (Claude Code, Codex, Cursor, Copilot CLI, etc.) and human contributors working in this repository. `CLAUDE.md` is a symlink to this file.

## Project shape

`clean-shortcuts` is a Swift package that compiles Apple Shortcuts from a result-builder DSL into signed `.shortcut` files. The library targets macOS 13+, iOS 16+, tvOS 16+, watchOS 9+. The CLI binary is macOS-only because it shells to `/usr/bin/shortcuts sign`.

```
Sources/
├── Shortcuts/              core library — DSL, encoders, compiler, discovery, decompiler, validator
│   ├── Actions/            one file per typed action
│   └── Internal/           private helpers
├── ShortcutsMacros/        @Action attached macro (compiler plugin)
├── ShortcutsJSON/          declarative JSON frontend (Foundation only)
├── ShortcutsYAML/          declarative YAML frontend (uses jpsim/Yams)
└── ShortcutsCLI/           `shortcuts-cli` binary

Tests/ShortcutsTests/       swift-testing test target; 71 tests, plus E2E gated on CLEAN_SHORTCUTS_E2E=1
Documentation/              FILE-FORMAT.md, PARAM-TYPING-CONTRACT.md, etc.
```

## Build / test

```
swift build -c release           # build everything
swift test                       # 71 tests, ~0.1s
swift test --parallel
.build/release/shortcuts-cli ... # CLI binary
```

E2E validation runs against the live Shortcuts.app runtime and is gated:

```
CLEAN_SHORTCUTS_E2E=1 swift test --filter ValidationTests
```

## Adding a typed action

Most contributors will be adding a new typed wrapper for some `is.workflow.actions.*` identifier. Use this checklist:

1. **Discover the parameter shape.** Run `.build/release/shortcuts-cli discover --emit json --output catalog.json` and find your identifier. Compare against a real installed shortcut by running `info` on a sample `.shortcut` to see the exact `WFWorkflowActionParameters` shape Shortcuts.app produces.
2. **Create the file.** Add `Sources/Shortcuts/Actions/MyAction.swift`. Mirror an existing file with similar parameter shape (e.g. `Notification.swift` for one-required, one-optional Value).
3. **Honor the parameter-typing contract.** Every field in `parameters()` must use exactly one of the slots in [the parameter typing contract](Documentation/PARAM-TYPING-CONTRACT.md). Don't invent a new slot — extend `Encoding.swift` if you truly need one.
4. **Add a typing test.** Append a case to `Tests/ShortcutsTests/ParamTypingTests.swift`. One `@Test` per action, using `expectValueShape` / `expectDictShape` / `expectBool` / `expectInt` / `expectString` helpers.
5. **Add a snapshot test.** Append to `Tests/ShortcutsTests/CompileTests.swift` to pin the full plist shape against expected values.
6. **Update the README action table** with the new action name.
7. **Run `swift test`** — all 71+ tests must pass.

## Adding a declarative-document action keyword

The YAML + JSON frontends share `Sources/Shortcuts/DocumentBuilder.swift`. To expose your new action there, add a `case "my-action":` branch in `buildAction(raw:outputs:)` that pulls fields from the body dict and constructs your typed action.

Add corresponding tests in both `YAMLFrontendTests.swift` and `JSONFrontendTests.swift`.

## Hard rules

- **Don't add new third-party dependencies** without checking that the same capability isn't already in Foundation or another Apple-owned package. The current third-party deps are:
  - `swiftlang/swift-syntax` (Apple-affiliated, required for macros)
  - `apple/swift-argument-parser` (Apple)
  - `apple/swift-testing` (Apple, transitional — bundled in Swift 6 but CLT 6.3 is missing the runtime dylib)
  - `jpsim/Yams` (de-facto Swift YAML; Apple uses it in SwiftPM itself, SwiftLint, SwiftFormat)
- **Don't break the wire format.** Generated plists must match what Shortcuts.app writes. If `swift test` snapshot tests fail, the change is wrong unless the snapshot itself was intentionally updated.
- **No emojis in code or generated output** unless explicitly requested.
- **No marketing copy in comments.** Comments explain the *why* of non-obvious code, never restate what the code does.
- **No personal references** of any kind (author handles, machine names, project names from unrelated work). Repo is public-facing.
- **No version-control checks of Apple's Gallery files.** They're Apple-authored content. `Tests/ShortcutsTests/Fixtures/*.wflow` is gitignored; tests find Gallery on-disk at runtime.

## Code style

- Swift 6.0+ language mode. Strict concurrency on by default.
- `public` API everywhere a consumer might call from outside the package.
- `internal` for cross-file helpers within `Shortcuts`. `private` for single-file scope.
- `Sendable` everywhere — every public type must be Sendable.
- One type per file. Filename matches the primary type.
- `// MARK: -` only when a single file has 2+ public types or 3+ helper sections. Don't add MARK to short files.

## Testing philosophy

- **Snapshot tests** pin behavior. They're the regression safety net.
- **Property tests** (`PropertyTests.swift`) cover encoder invariants — round-tripping, UTF-16 offset correctness, etc.
- **Determinism tests** ensure the same input produces the same plist (modulo UUIDs).
- **Macro expansion tests** pin the source generated by `@Action`.
- **Gallery round-trip tests** verify the decompiler handles every real Apple-authored `.wflow` on this Mac.
- **E2E `ValidationTests`** is the only test that proves a compiled shortcut actually runs in Shortcuts.app. Gated because it mutates user state.

## CI

`.github/workflows/test.yml` runs on `macos-14` and executes `swift test --parallel` plus a CLI smoke test (`demo` + `discover`). Keep CI green or the PR doesn't merge.

## Cutting a release

1. Update `Sources/Shortcuts/Version.swift` and `CHANGELOG.md`.
2. Tag: `git tag 0.x.0 && git push --tags`.
3. GitHub release: include the CHANGELOG entry and a one-line install snippet.

## Don't

- Don't push to `main` without a clean `swift test` run on macOS.
- Don't add YAML or JSON parsing logic outside the relevant frontend module — those modules are the boundary.
- Don't introduce `@objc` or NSObject inheritance. The library is value-type-first.
- Don't shell to external tools from the library target. Shelling is allowed in `Compiler.swift` (for `/usr/bin/shortcuts sign`), `Validation.swift` (for `open` + `shortcuts run`), `Discovery.swift` (for `/usr/bin/dyld_info`), and the CLI — that's the entire allowlist.

---
> Source: [damionrashford/clean-shortcuts](https://github.com/damionrashford/clean-shortcuts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
