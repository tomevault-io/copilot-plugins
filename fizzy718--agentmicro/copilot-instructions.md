## agentmicro

> - `Sources/CodexBar`: Swift 6 menu bar app (usage/credits probes, icon renderer, settings). Keep changes small and reuse existing helpers.

# Repository Guidelines

## Project Structure & Modules
- `Sources/CodexBar`: Swift 6 menu bar app (usage/credits probes, icon renderer, settings). Keep changes small and reuse existing helpers.
- `Sources/AgentMicro`: AgentMicro's focused menu bar executable. Keep the V1 surface Codex-only and read-only.
- `Tests/CodexBarTests`: XCTest coverage for usage parsing, status probes, icon patterns; mirror new logic with focused tests.
- `Tests/AgentMicroTests`: focused AgentMicro model and state-reducer tests. Prefer these seams over live AppKit tests.
- `docs/agentmicro`: AgentMicro product definition, V1 specification, roadmap, implementation log, and backlog.
- `Scripts`: build/package helpers (`package_app.sh`, `sign-and-notarize.sh`, `make_appcast.sh`, `build_icon.sh`, `compile_and_run.sh`). Release wrappers call `Scripts/mac-release`, which resolves `MAC_RELEASE_TOOL` or the shared `agent-scripts` checkout.
- `docs`: release notes and process (`docs/RELEASING.md`, screenshots). Root-level zips/appcast are generated artifacts—avoid editing except during releases.

## Build, Test, Run
- AgentMicro dev run: `AGENTMICRO_BUILD_ONLY=1 swift run --disable-automatic-resolution AgentMicro`.
- AgentMicro focused tests: `AGENTMICRO_BUILD_ONLY=1 swift test --disable-automatic-resolution --filter AgentMicro`.
- `AGENTMICRO_BUILD_ONLY=1` excludes the upstream CodexBar app targets and Sparkle artifact from the build graph; pair it with `--disable-automatic-resolution` so the upstream `Package.resolved` remains unchanged.
- Dev loop: `./Scripts/compile_and_run.sh` kills old instances, builds, packages, relaunches `CodexBar.app`, and confirms it stays running; add `--test` for the sharded full suite.
- Quick build/test: `swift build` (debug) or `swift build -c release`; `make test` for the sharded full suite.
- Package locally: `./Scripts/package_app.sh` to refresh `CodexBar.app`, then restart with `pkill -x CodexBar || pkill -f CodexBar.app || true; cd /Users/steipete/Projects/codexbar && open -n /Users/steipete/Projects/codexbar/CodexBar.app`.
- Release flow: `./Scripts/release.sh`; app metadata lives in `.mac-release.env`, repo build/signing stays in `Scripts/sign-and-notarize.sh`, and validation steps live in `docs/RELEASING.md`.

## Coding Style & Naming
- Enforce SwiftFormat/SwiftLint: run `swiftformat Sources Tests` and `swiftlint --strict`. 4-space indent, 120-char lines, explicit `self` is intentional—do not remove.
- Favor small, typed structs/enums; maintain existing `MARK` organization. Use descriptive symbols; match current commit tone.

## Testing Guidelines
- Add/extend XCTest cases under `Tests/CodexBarTests/*Tests.swift` (`FeatureNameTests` with `test_caseDescription` methods).
- Add AgentMicro-specific coverage under `Tests/AgentMicroTests`.
- Swift Testing: prefer backticked sentence names; no camelCase.
- Model names in tests/code: released models or clearly fictitious names only; never expose unreleased names.
- Always run `make test` before handoff; add focused `swift test --filter ...` runs for parser/provider fixes when possible.
- After any code change, run `make check` and fix all reported format/lint issues before handoff.
- Prefer CLI/focused tests over app-bundle live tests when behavior can be verified without relaunching CodexBar.
- Never run tests/checks or ad-hoc validation that can display macOS Keychain prompts. Live provider probes, browser-cookie imports, `codexbar usage` against real accounts, and real SecItem reads must be explicitly requested; otherwise use parser tests, stubs, test stores, or `KeychainNoUIQuery`.
- macOS CI is brittle around headless AppKit status/menu tests. Prefer covering menu behavior through stable state/model seams (`MenuDescriptor`, `ProvidersPane`, `CodexAccountsSectionState`, etc.) instead of constructing live `NSStatusBar`/`NSMenu` flows unless the AppKit wiring itself is the thing under test.

## Commit & PR Guidelines
- Commit messages: short imperative clauses (e.g., “Improve usage probe”, “Fix icon dimming”); keep commits scoped.
- PRs/patches should list summary, commands run, screenshots/GIFs for UI changes, and linked issue/reference when relevant.

## AgentMicro Product Documentation
- `docs/agentmicro/README.md` is the index for AgentMicro product truth. Read the relevant product docs before changing scope, task-state semantics, menu behavior, privacy boundaries, or roadmap commitments.
- Update `docs/agentmicro/V1_SPEC.md` when V1 behavior or acceptance criteria change, and update `docs/agentmicro/PRODUCT.md` when the product definition or long-term boundary changes.
- Record every completed product, architecture, implementation, documentation, or release change in `docs/agentmicro/PROJECT_LOG.md`. Include the date, impact, verification, and known limitations. Do not use the upstream root `CHANGELOG.md` for AgentMicro work.
- Put new ideas and deferred requirements in `docs/agentmicro/BACKLOG.md` before expanding the active version. Promote work into `docs/agentmicro/ROADMAP.md` only when its milestone and entry conditions are explicit.
- Keep `docs/agentmicro/CAPABILITY_MAP.md` current when an upstream project or reuse decision changes. Distinguish implemented facts from plans and heuristics.
- Product documentation is a required handoff step, not optional cleanup. Before ending any AgentMicro product or behavior task, explicitly review every file below and update each affected layer:
  - `PRODUCT.md`: current promise, user problem, product principles, and long-term boundary.
  - `V1_SPEC.md`: current UI behavior, state semantics, data rules, refresh policy, and acceptance criteria.
  - `README.md`: current implementation summary and navigation; it must describe what the installed product does now.
  - `ROADMAP.md`: active milestone scope, status names, exit conditions, and sequencing when they change.
  - `BACKLOG.md`: deferred ideas, unresolved limitations, external blockers, and follow-up research.
  - `CAPABILITY_MAP.md`: upstream evidence and reuse decisions when they change.
  - `PROJECT_LOG.md`: dated record of what actually changed, its impact, verification, and remaining limitations.
- Do not consider the documentation pass complete after updating only `V1_SPEC` and `PROJECT_LOG`. Search all `docs/agentmicro` files for superseded state names, timings, colors, UI fields, retention windows, milestone claims, and product promises; update current-truth documents or clearly identify text as historical.
- Keep historical `PROJECT_LOG` entries intact unless they are factually malformed. Current truth belongs in `PRODUCT`, `V1_SPEC`, `README`, `ROADMAP`, `BACKLOG`, and `CAPABILITY_MAP`; the log records how the product arrived there.
- In the final handoff, name the product documents updated and call out any known documentation gap. If no product document changed, state why the task had no product, behavior, scope, limitation, or roadmap impact.

## Agent Notes
- Use the provided scripts and package manager (SwiftPM); avoid adding dependencies or tooling without confirmation.
- Menu bar automation: capture the target screen first and verify the CodexBar icon is visibly onscreen. Reject `click-extra` success when coordinates fall outside display bounds; hidden menu extras are not click proof.
- Validate UI/runtime behavior against the freshly built bundle; restart via the pkill+open command above to avoid running stale binaries.
- To guarantee the right bundle is running after a rebuild, use: `pkill -x CodexBar || pkill -f CodexBar.app || true; cd /Users/steipete/Projects/codexbar && open -n /Users/steipete/Projects/codexbar/CodexBar.app`.
- For CLI-testable provider/parser/settings behavior, use CLI/focused tests instead of `Scripts/package_app.sh` or `./Scripts/compile_and_run.sh`.
- Run `./Scripts/compile_and_run.sh` only when UI/runtime behavior needs bundle-level validation; it builds, tests, packages, relaunches, and verifies the app stays running.
- Widget/Tahoe UI issues: use Parallels macOS VM plus screenshots/clicks for autonomous verification.
- Release script: keep it in the foreground; do not background it—wait until it finishes.
- AgentMicro application icon: keep the six lights as independent layers in `AgentMicro.icon`; `Scripts/build_agentmicro_icon.sh` must use Xcode `actool` and package both `Assets.car` and the ICNS fallback. Set `XCODE_APP=/Applications/Xcode-beta.app` when the host OS requires the beta toolchain; do not regress to a Default-only `ictool` export.
- CodexBar Sparkle release key: use `.mac-release.env` `MAC_RELEASE_SIGNING_KEY_FILE`, the legacy `AGCY8w5vHirVfGGDGc8Szc5iuOqupZSh9pMj/Qs67XI=` key. AgentMicro must use its own `AGENTMICRO_PUBLIC_ED_KEY` and dedicated Keychain account/private key; never reuse the CodexBar or VibeTunnel key.
- Swift concurrency: treat sibling `async let` tasks as a review red flag when one child is required and another is optional/best-effort. Prefer sequential awaits or a drained `withThrowingTaskGroup` that surfaces required failures and explicitly contains optional failures; crash stacks mentioning `swift_task_dealloc` or `asyncLet_finish_after_task_completion` should trigger an audit of nearby `async let` usage.
- Prefer modern SwiftUI/Observation macros: use `@Observable` models with `@State` ownership and `@Bindable` in views; avoid `ObservableObject`, `@ObservedObject`, and `@StateObject`.
- Favor modern macOS 15+ APIs over legacy/deprecated counterparts when refactoring (Observation, new display link APIs, updated menu item styling, etc.).
- Keep provider data siloed: when rendering usage or account info for a provider (Claude vs Codex), never display identity/plan fields sourced from a different provider.***
- Claude CLI status line is custom + user-configurable; never rely on it for usage parsing.
- Cookie imports: default Chrome-only when possible to avoid other browser prompts; override via browser list when needed.

---
> Source: [fizzy718/AgentMicro](https://github.com/fizzy718/AgentMicro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
