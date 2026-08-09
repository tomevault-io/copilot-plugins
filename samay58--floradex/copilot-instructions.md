## floradex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Floradex is a SwiftUI iOS app (iOS 26 deployment target, Xcode 26) that identifies plants from photos and saves pixel-art "dex" entries with care details. Naming quirk: the Xcode project, app target, and Swift module are all named `plantlife`, but the only scheme is `floradex`. Use `-scheme floradex` in xcodebuild commands and `@testable import plantlife` in tests.

## Commands

```bash
# Kit logic tests (Swift Testing, runs on macOS, no simulator; the fastest check)
cd FloradexKit && swift test

# Build for simulator
xcodebuild -project plantlife.xcodeproj -scheme floradex -sdk iphonesimulator build

# App unit tests (destination must name an installed simulator; check `xcrun simctl list devices`)
xcodebuild -project plantlife.xcodeproj -scheme floradex test \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:plantlifeTests -parallel-testing-enabled NO

# Run a single test class or method
xcodebuild -project plantlife.xcodeproj -scheme floradex test \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:plantlifeTests/SwiftDataDexStoreTests/testDeleteRetiresTheNumberForever
```

Always scope simulator test runs with `-only-testing:plantlifeTests`: the scheme's test action also includes `plantlifeUITests`, which is untouched Xcode template scaffolding until phase 7 makes it real.

If XcodeBuildMCP tools are available, prefer them over raw xcodebuild.

Machine notes:
- `xcode-select` points at CommandLineTools, so prefix `xcodebuild`/`xcrun`/`swift` with `DEVELOPER_DIR=/Applications/Xcode-beta.app/Contents/Developer` or they fail with a "requires Xcode" error.
- The iOS simulator runtime disk image can be absent (purged under disk pressure on 2026-07-02). If `xcrun simctl runtime list` shows no images, reinstall with `xcodebuild -downloadPlatform iOS` (about 8GB) before any simulator work. Kit tests are unaffected.

API keys are development environment variables (`KINDWISE_API_KEY`, `PLANTNET_API_KEY`, `OPENAI_API_KEY`) resolved through `CredentialBroker` at request time; there is no xcconfig path and nothing key-shaped in the repo. `FLORADEX_FIXTURES=1` runs the app with no keys at all.

The shared scheme (`plantlife.xcodeproj/xcshareddata/xcschemes/floradex.xcscheme`) carries `FLORADEX_FIXTURES`, `FLORADEX_AUTORUN`, `FLORADEX_TAB`, and `FLORADEX_ENTRY` as disabled environment variables; enable them in the scheme editor for Xcode runs, or pass them as `SIMCTL_CHILD_`-prefixed variables when launching via `simctl launch`. `plantlife/Shared/DebugFlags.swift` is the single reader for these flags. API keys stay out of that file deliberately (it is tracked).

## Architecture (rewrite in progress, phases 2 through 6 landed)

Read `docs/rewrite-research/floradex-rewrite-spec.md` before any structural change; it defines the architecture, the 8-phase plan, and what is deliberately deferred. `docs/rewrite-research/floradex-modern-ios-research.md` holds the platform decisions with sources. `docs/rewrite-research/WHERE-WE-LEFT-OFF.md` is the session handoff doc: current state, verify-before-building commands, and decisions parked with the user. Read it before resuming rewrite work and keep it updated when a session materially advances a phase. Branch: `rewrite/foundation`.

**The seam**: `FloradexKit/` is a local Swift package (no SwiftUI/UIKit) linked into the app target. Everything is Swift 6: the app target builds with `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor` (deliberately off-main types are marked `nonisolated`); the test target is Swift 6 without the default (XCTestCase's nonisolated lifecycle forbids it) and isolates test classes explicitly. Domain logic, policies, the hero-loop reducer, the orchestrator actor, provider API clients, and the fixture catalog live there. Verify with `cd FloradexKit && swift test` (runs on macOS, no simulator). Boundary rule: needs SwiftUI or live hardware → `plantlife/`; otherwise → the Kit.

**Hero loop** (`plantlife/Features/`): `CaptureHomeView` (Identify tab) → `CameraSession` actor (pre-warm, responsive capture) → `CaptureFlowModel` (`@Observable`, executes effects only; all sequencing lives in the Kit's `IdentificationFlowReducer`) → `IdentificationOrchestrator` actor drives the data-driven `EscalationPolicy` over providers (Kindwise, Pl@ntNet, OpenAI vision reasoner) → staged `RevealCard` with undo window → commit assigns the dex number (monotonic, never reused; deletes leave gaps). `FLORADEX_FIXTURES=1` + `FLORADEX_AUTORUN=1` run the whole loop on a simulator with canned providers and no keys.

**Credentials**: all provider calls resolve keys through `CredentialBroker` at request time (env vars `KINDWISE_API_KEY`, `PLANTNET_API_KEY`, `OPENAI_API_KEY` in development; a Cloudflare Workers + App Attest proxy broker replaces it before release). Missing keys throw typed `.credentialMissing` errors; there are no silent no-ops and no keys in the binary.

**Persistence (v2, phase 5)**: `plantlife/Models/FloradexSchema.swift` holds both versioned schemas; `FloradexMigrationPlan` migrates v1 stores in place (numbers freeze, gaps become ledger tombstones, image blobs export to disk). `SwiftDataDexStore` implements the Kit's `DexStore` seam over `DexEntryV2`/`SpeciesRecord`/`DexLedger` and is the schema's single writer. Media lives under `MediaLocations.root` keyed by each entry's `mediaID` through the Kit's `MediaPathPolicy`/`FileMediaStore`. Collection surfaces are `Features/Dex/DexGridView` (grid, list escape hatch, search, sort, batch delete) and `Features/Entry/EntryDetailView`; the root is a native `TabView` in `PlantLifeApp.swift` (`FLORADEX_TAB=dex` preselects the collection in DEBUG). No legacy surfaces remain.

**Tests**: Kit logic in Swift Testing (101 tests, `swift test`); app unit tests in XCTest on simulator (`-only-testing:plantlifeTests`, use `-parallel-testing-enabled NO`): the seeded v1-to-v2 migration test plus the SwiftData store suite. The 16-case fixture corpus in `FloradexKitFixtures` replays through the real escalation engine and orchestrator.

## Rewrite status and rules

- Done: phase 2 (dead code wave 1, test repair, iOS 26 crash fixes), phase 3 (project wiring, deployment target 26.0), phase 4 (Kit orchestrator + provider clients + hero loop UI), phase 5 (v2 schema + migration, new dex/entry surfaces, native TabView root, all legacy deleted), phase 6 (trust and correction states, Swift 6 flip via `scripts/flip_swift6.rb`), and the phase 8 visual identity (2026-07-02 design session: herbarium direction, token layer, all surfaces restyled, app icon).
- Next: phase 7 (fixture assets, Maestro/XCUITest), phase 8 remainder (offline queue, proxy scaffold in `proxy/`, on-device frame pacing).
- Design language: `plantlife/UI/FloradexTheme.swift` is the token layer and carries the register rule (words serif on paper, dex numbers in the bundled Departure Mono pixel face, machined motion). New surfaces use its tokens, never ad hoc values; the pixel face renders dex numbers and nothing else.
- Never hand-edit `project.pbxproj`; use `scripts/wire_floradexkit.rb` as the pattern (xcodeproj gem, checkpoint commit, line-by-line diff review, green build) for any further project mutations.
- Warning budget: zero project warnings as of the app-icon fill on 2026-07-02; the only remaining build-log line is `appintentsmetadataprocessor` toolchain noise (`docs/rewrite-research/warning-baseline.md` started at 140). The count stays at zero, and new code merges with zero warnings.
- Path note: `/Users/samaydhawan/floradex` is a symlink to this checkout, not a separate worktree.

---
> Source: [samay58/floradex](https://github.com/samay58/floradex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
