## pi-remote-ios

> This file is the entry point for any agent working on `pi-remote-ios`. Read it first. For deeper specs see `Systems.md`, `PLAN.md`, and `docs/`. `CLAUDE.md` is a symlink to this file, so Claude Code and other agents read identical instructions — edit only `AGENTS.md`.

# AGENTS.md — Project orientation

This file is the entry point for any agent working on `pi-remote-ios`. Read it first. For deeper specs see `Systems.md`, `PLAN.md`, and `docs/`. `CLAUDE.md` is a symlink to this file, so Claude Code and other agents read identical instructions — edit only `AGENTS.md`.

## What this project is

A two-half system that lets a paired iOS app control an active Pi (`@earendil-works/pi-coding-agent`) terminal session over WebSocket:

1. **TypeScript Pi extension** (`extension/`) — loaded into a running `pi` process via the project's `.pi/settings.json`. Exposes a `/remote` slash command, runs an HTTP/WebSocket server, owns auth + protocol + Pi event mapping.
2. **iOS app** (`Pi/`) — native SwiftUI app built on **The Composable Architecture (TCA)**. Connects to the extension over WebSocket, pairs once with a 6-digit code, then drives the session: send prompts, abort, compact, observe streaming assistant output and tool calls.

User flow:

```
$ pi                           # in repo root, loads extension via .pi/settings.json
> /remote start                # starts ws server on the Pi host's local network.
                               # Prints status, 6-digit code, and a QR code encoding a
                               # pi-remote://pair?url=...&code=... deep link.

iPhone: scan QR from Camera or in-app → auto-fills + auto-pairs
                               # extension returns one-time device token
                               # iOS stores the token in Keychain for future reconnects

iPhone: type prompt → Send     # round-trip through WebSocket → Pi → LLM → streaming back
                               # requires the iPhone and Pi host on the same trusted network
```

`/remote start` means "running and ready to pair." There is intentionally no separate `/remote pair` command.

## Repository layout

```
pi-remote-ios/
├── Project.swift           Tuist source of truth for iOS targets and settings
├── Tuist.swift             shared Tuist configuration
├── Tuist/Package.swift     external Swift package dependencies
├── extension/              TypeScript Pi extension (server side)
│   ├── index.ts            thin entry point
│   ├── command.ts          /remote command handler + UI notifications
│   ├── config.ts           defaults + env overrides
│   ├── runtime/            RemoteControlRuntime (start/stop/status/rotate)
│   ├── server/             HTTP + WebSocket upgrade lifecycle
│   ├── connections/        RemoteConnection (socket lifecycle) + RemoteSession (auth + commands)
│   ├── auth/               DeviceTrust + PairingManager + TokenStore + crypto
│   ├── protocol/           wire envelopes + ProtocolDecoder + capabilities + error codes
│   ├── pi/                 PiRemoteAdapter (seam around ExtensionAPI) + sanitizer + editPayload (unified-diff extraction/synthesis)
│   ├── events/             RemoteEventStream (Pi event subscription + broadcast)
│   ├── questions/          remote_question tool + reconnect-safe QuestionBroker
│   ├── session/            structured phase monitor + run evidence collector
│   ├── tunnel/             dormant legacy Cloudflare tunnel implementation
│   └── logging/            structured logger
│
├── test/                   Node test suite for the extension (decoder/auth/integration)
│
├── docs/
│   ├── PROTOCOL.md         full wire protocol spec
│   ├── SECURITY.md         threat model + security invariants
│   ├── SWIFT_CLIENT_NOTES.md  iOS integration notes
│   └── API.md              short index
│
├── Pi/                     iOS app sources and tests
│   ├── PiTests/            Swift Testing coverage for pure app policies
│   ├── PiUITests/          XCTest UI coverage for live release acceptance
│   └── Pi/                 app sources (auto-included via synced root group)
│       ├── PiApp.swift     @main, instantiates the AppFeature Store
│       ├── DesignSystem/   AppColors / AppFonts / AppSpacing — single source of truth
│       ├── Components/     reusable views and the native Canvas Pi logo animation
│       ├── Core/
│       │   ├── Background/ ContinuedProcessingClient for minimized run monitoring
│       │   ├── Networking/ RemoteSocketClient (hand-rolled dependency; see comment)
│       │   ├── Notifications/ privacy-safe local completion/question alerts
│       │   ├── Persistence/CredentialsStore (Keychain-backed token + URL + device)
│       │   ├── Protocol/   wire-format types (mirror of extension/protocol)
│       │   ├── Model/      domain types (transcript, state snapshot, PairingDeepLink)
│       │   └── Extensions/ binding helpers
│       └── Features/
│           ├── AppFeature.swift       single TCA reducer that owns the session domain
│           ├── AppRootView.swift      routes by AppFeature.State.screen
│           ├── Pairing/               PairingView + PairingScannerSheet
│           ├── Dashboard/             DashboardView + DashboardHeader + Composer + Transcript/
│           ├── Reconnect/             ReconnectView
│           └── Settings/              SessionSettingsView + EnvelopeLogRow
│
├── AGENTS.md               this file — project orientation, rules, conventions
├── CLAUDE.md               symlink → AGENTS.md (do not edit directly)
├── Systems.md              expanded system prompt for agents
├── PLAN.md                 architecture plan + phase status
├── DESIGNER_BRIEF.md       product/design brief for the iOS app
└── README.md
```

## TypeScript extension

### Module map

| Module | Owns |
|---|---|
| `RemoteControlRuntime` | `start`/`stop`/`getStatus`/`rotateTrust` semantics; the invariant that "/remote start" means running + ready to pair. |
| `DeviceTrust` | Pairing window, code validation, token issue/verify, trusted device list, revocation. Wraps internal `PairingManager` + `TokenStore`. |
| `ProtocolDecoder` | Decodes raw WebSocket frames into typed `ClientMessage`. Validates protocolVersion, type, payload shape. |
| `RemoteConnection` | One client's WebSocket lifecycle only — hello, frame receive, heartbeat, auth timeout, close cleanup. Delegates protocol to `RemoteSession`. |
| `RemoteSession` | One client's auth state + authenticated command dispatch (`interaction.answer`, `session.getState`, `session.history`, `session.prompt`, `session.abort`, `session.compact`, `session.listModels`, `session.listSkills`, `session.setModel`, `session.setThinkingLevel`, `connection.ping`). |
| `RemoteServer` | HTTP + WebSocket upgrade server, connection registry, sequence numbers, authenticated-client broadcast. |
| `PiRemoteAdapter` | Seam around Pi's active session operations. Owns state/history, prompt delivery, model/thinking changes, abort, compact, and execution context. Registration modules use only Pi's command, tool, or event registration surfaces. |
| `RemoteEventStream` | Subscribes to Pi events (`message_*`, `tool_execution_*`, `agent_*`, …), maps to stable `pi.*` types, broadcasts only to authenticated clients, drives structured phase changes, and emits run completion evidence after `agent_settled`. Also attaches edit/write diffs to matching `pi.tool_end` events. |
| `editPayload` | Pure helper: `extractEditPayload(toolResultEvent)` lifts Pi's own diff for `edit`, `synthesizeWriteDiff(toolResultEvent)` builds an all-added `/dev/null` diff for `write`. Both bypass the sanitizer's 20K-char cap on the `diff` field so the iOS app can render whole diffs regardless of size. |
| `QuestionBroker` | Owns one bounded pending `remote_question`, restores it after token reconnect, and resolves answer, abort, timeout, or server stop. |
| `RemoteSessionMonitor` | Tracks `idle`, `running`, `queued`, `compacting`, `needsInput`, `completed`, and `failed` phases. |
| `RunEvidenceCollector` | Accumulates bounded command outcomes, tool failures, and changed-file statistics until `agent_settled`. |
| `CloudflareTunnel` | Legacy tunnel implementation retained in source. `/remote start` does not activate it. |

### Wire protocol (summary; full spec in `docs/PROTOCOL.md`)

- WebSocket at `/v1/ws`, JSON envelopes, `protocolVersion: 1`.
- Server envelope: `{ id?, seq, type, protocolVersion, timestamp, payload }`.
- Client envelope: `{ id?, type, protocolVersion, payload }`.
- Connection state machine: connect → `server.hello` → `auth.pair.request` (with code) **or** `auth.token` → `auth.*.success` + `state.snapshot` → live commands/events.
- Stable `response.ok` shape: `{ ok, command, result }`.
- State-changing commands emit `state.changed` after `response.ok`.
- Pi events come through as `pi.<name>` with payload `{ source: "pi", name, data }`.
- iOS requires `pi.agent_settled` in server capabilities and rejects stale loaded extensions with `/reload` guidance.
- Close codes: `1001` server shutdown, `4000` heartbeat timeout, `4001` auth timeout.

### Security invariants

- `/remote start` binds to `0.0.0.0` and advertises the host's LAN address.
- Unauthenticated clients can only receive `server.hello`, send `connection.ping`, or auth.
- Pi event broadcasts go only to authenticated connections.
- Pairing code expires after 2 minutes; 5 failed attempts disable the code; successful pair disables the code.
- Tokens are high-entropy and returned once. Server stores only `sha256(token + serverSecret)`.
- The extension stores trusted-device hashes in memory; the iOS app stores its issued token in Keychain.
- Sanitizer caps payloads at 20K-char strings, 200-item arrays, depth 8; redacts known secret keys.

### Running

```
npm install
npm run typecheck     # tsc --noEmit
npm test              # node --test test/*.test.mjs (66 tests)

pi                    # from repo root, loads extension via .pi/settings.json
> /remote start            # local network; iPhone and Pi host must share trusted Wi-Fi
                           # logs are silent by default — pass --logs to see
                           # connect/disconnect/auth events for debugging
> /remote start --host 127.0.0.1  # Simulator-only loopback
> /remote start --logs     # enable structured JSON logs for this session
```

## iOS app

### Architecture

**Single TCA reducer.** `AppFeature` owns the entire session domain — WebSocket lifecycle, auth, transcript, state snapshot, reconnect with backoff, keepalive, command dispatch, log. Views are stateless `StoreOf<AppFeature>` consumers organized by feature folder.

We made an explicit decision to keep this as one reducer rather than per-feature reducers because every "feature view" reads/writes the same session state. Splitting would force state duplication + delegate-action plumbing for no benefit.

### Reducer surface

State (`AppFeature.State`):
- Form fields (bindable): `serverURL`, `pairingCode`, `deviceName`, `promptText`, `questionAnswerText`, `busyPromptDelivery`, `modelPickerPresented`, `settingsPresented`.
- Session data: `status`, `hello`, `device`, `token`, `sessionState`, `pendingQuestion`, `lastRun`, `lastSeq`, `lastError`, `connectedAt`, `currentRunStartedAt`, `log`, `transcript`, `pendingPromptRequests`, `backgroundRunMonitoringID`, `backgroundRunPromptRequestID`, `copiedResponseID`, `availableModels`, `availableSkills`.
- Reconnect bookkeeping: `reconnectAttempt`, `userInitiatedDisconnect`.
- Derived: `isAuthenticated`, `canPair`, `screen` (used by `AppRootView` to pick `pairing | dashboard | reconnecting`).

Actions (named after the user action that produces them):
- User: `connectButtonTapped`, `disconnectButtonTapped`, `pairButtonTapped`, `sendPromptButtonTapped`, `questionOptionButtonTapped(requestId:answer:)`, `questionCustomAnswerButtonTapped`, `retryPromptButtonTapped(id:)`, `copyResponseButtonTapped(id:text:)`, `skillSuggestionTapped(name:)`, `abortButtonTapped`, `compactButtonTapped`, `getStateButtonTapped`, `retryReconnectButtonTapped`, `forgetDeviceButtonTapped`, `ellipsisButtonTapped`, `sheetDismissed`, `deepLinkOpened(URL)`, `refreshModelsRequested`, `modelPickerButtonTapped`, `modelSelected(provider:modelId:)`, `thinkingLevelSelected(level:)`.
- Lifecycle: `onAppear` (load persisted credentials + auto-resume), `scenePhaseDidActivate` (foreground → force reconnect if we hold a token and aren't authenticated).
- Effect responses: `backgroundRunMonitoringStarted`, `socketEvent(RemoteSocketClient.Event)`, `keepaliveTick`, `reconnectFire`, `sendSucceeded`, `sendFailed`.
- Plus `binding(BindingAction<State>)` for form fields.

`deepLinkOpened(URL)` is dispatched by `RootView`'s `.onOpenURL` when the OS routes a `pi-remote://pair?url=...&code=...` URI into the app (Camera-app scan, Safari, or the in-app scanner). The reducer parses via `PairingDeepLink.parse`, fills `serverURL` + `pairingCode`, sets a transient `autoPairOnReady` flag, and runs `connectEffect`. `handleServerHello` consumes the flag and fires `auth.pair.request` automatically. Already-authenticated state is a no-op (the user must `forgetDevice` first to re-pair).

Cancel IDs: `socket`, `keepalive`, `reconnect`.

### Dependencies

Injected via TCA's `@Dependency`:

- `\.remoteSocket` → `RemoteSocketClient` (hand-rolled; **not** `@DependencyClient` — that macro silently substitutes parameter defaults for actual arguments on iOS 26 and burned us once). Closures: `open(String) → AsyncStream<Event>`, `send(String) async throws → Void`, `close() async → Void`. Live impl wraps `URLSessionWebSocketTask` in a private actor.
- `\.credentialsStore` → `CredentialsStore`. Keychain-backed (KeychainSwift, `.afterFirstUnlock`) JSON of `{token, serverURL, device}`. Saved on `auth.pair.success` and `auth.token.success`, cleared on forget-device and on auth-failure rejection. Loaded by `onAppear` to drive cold-launch resume.
- `\.clipboard` → `ClipboardClient`. Copies completed assistant responses through the system pasteboard and becomes a no-op in tests unless overridden.
- `\.continuedProcessing` → `ContinuedProcessingClient`. Submits one iOS 26 `BGContinuedProcessingTask` for a user-started Pi run, updates its generic phase, and ends it with the run. The system provides the Live Activity.
- `\.localNotifications` → `LocalNotificationClient`. Requests alert permission in the context of the first monitored prompt and sends generic needs-input/completion alerts without session content.
- `\.continuousClock` → `ContinuousClock` for keepalive (20s) and backoff (`0.5/1/2/5/10s`, capped at 8 attempts ≈ 30s).
- `\.uuid` → `UUIDGenerator` for request IDs and entry IDs (overridden to `.incrementing` in tests).
- `\.date` → `DateGenerator` for log timestamps and `connectedAt`.

### Design system

`DesignSystem/` is the single source of truth. Components reference these — no hardcoded hex/sizes:

- `AppColors` — semantic tokens (`background`, `surface`, `surfaceElevated`, `codeBackground`, `scrim`, `text`/`textSecondary`/`textTertiary`, `cardBorder`, `accent`/`onAccent`, `success`/`info`/`highlight`/`warning`/`error`/`thinking`). Every token is **adaptive light/dark** via `UIColor(dynamicProvider:)`; values come from the Pi Remote design spec (dark bg `#0C0C0D`, green accent `#5ED6A1` dark / `#0B7A5C` light). The splash keeps a forced dark scheme; everything else follows the system appearance.
- `AppFonts` — `Font.app*` extension (`appBody` 16, `appBodyBold`, `appTitle`, `appSheetTitle`, `appHeadline`, `appCallout`, `appCaption`, `appLabel`, `appMono*`).
- `AppSpacing` — base scale `xs`/`s`/`m`/`l`/`xl`/`xxl` + role tokens (`cardPadding`, `cardRadius`, `toolCardRadius` 14, `bubbleRadius` 18, `sheetRadius` 32, `buttonHeight`, `buttonRadius`, `pillRadius`, `chipRadius` 10). Cards are flat surfaces — no stroke borders except input fields and focus rings.

### Components and shared UI

`Components/` is feature-agnostic. Each file has a top-of-file docstring (when to use, parameters, example) and one or more `#Preview` macros covering meaningful states. Files include `Card`, `PrimaryButton`, `StatusDot`, `ScreenHeader`, `SectionLabel`, `DiagnosticRow`, `JumpToLatestButton`, `MarkdownText`, `ReconnectBanner`, `PiDiffView`, and `SessionActivityGlow`.

`PiDiffView` renders Pi's annotated diff format directly — one row per source line, gutter + colored prefix + per-row horizontal scroll for long content. We skip the gitdiff round-trip for edit/write tool diffs because Pi already produces a fully formatted, terminal-ready diff; the abstraction tax wasn't earning its keep and gitdiff's row layout had a long-line clipping interaction. `gitdiff` is still pinned in `Package.resolved` for any future surface that consumes standard unified-diff text.

`MarkdownText` renders markdown via `AttributedString(markdown:options:)` with `interpretedSyntax: .full` — iOS 26's block-level markdown rendering in `Text` covers lists, headings, code blocks, and inline emphasis. Falls back to plain text when streaming buffers end mid-token. Used by `AssistantMessage` for direct-canvas assistant output; user-typed prompts stay plain so unintentional `*` characters do not become italics.

`Components/AnimatedLogo/` contains `PiLogoAnimationView`, a native SwiftUI `Canvas` port of the pi.dev logo intro. It reuses the same 8×9 board, pieces, colors, and timing constants extracted from pi.dev; prefer it over GIF/MP4 assets for app UI.

The app icon is the live-session mark in `Pi/Design/AppIcon-LiveSession.svg`. Export its asset variants with `scripts/export-app-icon.sh`.

### Features

- `AppFeature.swift` + `AppRootView.swift` — root reducer + screen router. A user-started prompt also starts best-effort continued processing so the WebSocket can monitor Pi after minimization. iOS provides a system Live Activity, and needs-input/completion events can create generic local notifications. Swiping the app away or system expiration stops monitoring. `AppRootView` runs a local "reconnect escalation" timer: while `screen == .reconnecting` it renders the slim `ReconnectBanner` over the live Dashboard for the first ~1.2s (warm bounces resolve invisibly); if the reconnect drags past that, or if the transcript is empty (cold-launch resume — nothing meaningful to render under a banner), it promotes to the full `ReconnectView` card overlay.
- `Pairing/PairingView.swift` — host field, 6-digit code grid, status banner, pair CTA.
- `Dashboard/DashboardView.swift` + `DashboardHeader.swift` + `ModelPickerView.swift` + `Composer.swift` + `RunShelf.swift` + `ContextBanner.swift` + `QuestionCard.swift` + `SkillSuggestions.swift` + `Transcript/` — chat-first main screen with slash autocomplete, explicit Queue/Steer, blocking question cards, and completion evidence. The header shows a π avatar with a phase dot (or needs-input badge), `project · branch` title, and a `Model · Thinking` subtitle that opens the searchable Model & Thinking sheet (models list + thinking-level chips, hides unconfigured models). While Pi works, `RunShelf` pins above the composer with a pulsing dot, live elapsed timer (`currentRunStartedAt` in reducer state), and the Stop control; `ContextBanner` appears at ≥80% context usage with an inline Compact action. Busy sessions expose explicit Queue/Steer delivery. User prompts appear optimistically with sending, queued, sent, failed, and retry states. Consecutive tools collapse into an expandable activity group. Completed assistant responses expose Copy and Share actions. Automatic transcript following pauses when the user scrolls away and resumes through `JumpToLatestButton`. Aurora renders a full-screen edge glow while Pi works.
- `Reconnect/ReconnectView.swift` — overlay during auto-reconnect.
- `Settings/SessionSettingsView.swift` + `EnvelopeLogRow.swift` — the Session Inspector sheet: connection/session/trust diagnostics, a plain-language security section ("terminal-equivalent control … authenticated but not encrypted"), controls, and the envelope log.

### Visual direction

The "Pi Remote iOS Design Spec" (claude.ai/design project): adaptive light/dark, near-black `#0C0C0D` background with `#19191B`/`#242427` surfaces in dark, white with `#F4F4F2`/`#ECECEA` in light, green accent (`#5ED6A1` dark / `#0B7A5C` light), SF for body, SF Mono for paths/codes/timestamps. SF Symbols cover all icons. Chat-style transcript: user prompts in `surface` bubbles (18pt radius, 4pt tail corner), assistant prose directly on the canvas, tool events in flat 14pt-radius cards. The run shelf above the composer is the only "running" indicator chrome and owns the Stop control. System-visible surfaces (notifications, Live Activity) stay generic — never a prompt, path, branch, or file name.

### Building

```
tuist install
tuist generate --no-open
xcodebuild -workspace Pi.xcworkspace \
           -scheme Pi \
           -destination 'generic/platform=iOS Simulator' \
           build
```

`Project.swift` is the source of truth. It includes app files under `Pi/Pi/`, unit tests under `Pi/PiTests/`, UI tests under `Pi/PiUITests/`, and the asset catalog. Regenerate after manifest or file changes. Generated `Pi.xcodeproj`, `Pi.xcworkspace`, and `Derived/` paths are ignored.

The pre-Tuist project remains available in Git history. Do not restore or edit it for new source, setting, or dependency changes.

`Tuist/Package.swift` pins `ComposableArchitecture` 1.25.5, `KeychainSwift` 24.0.0, `gitdiff` 0.1.0, and `Aurora` 0.4.0. `Tuist/Package.resolved` preserves the existing transitive versions.

The `PiTests` target uses Swift Testing. Run it with `xcodebuild -workspace Pi.xcworkspace -scheme Pi -destination 'platform=iOS Simulator,name=iPhone 17 Pro' -skipMacroValidation test`.

The `PiUITests` target uses XCTest UI automation under the `PiE2E` scheme. `scripts/run-e2e.mjs` starts a real Pi RPC process. It checks pairing, prompts, completion, background return, cold launch, and reconnect. It resets the selected Simulator keychain and removes stale `com.techzy.Pi` installs. `scripts/verify-linux-e2e.mjs` checks the extension in `node:22-bookworm` through Docker.

## Conventions

### Swift code style

- **Indent**: 2 spaces. Tab width 2. Apply consistently.
- **Type naming**: no project prefix on generic primitives (`Card`, `PrimaryButton`, not `PiCard`/`PiCTA`). Design system uses an `App` prefix to avoid SwiftUI namespace clashes (`AppColors`, `Font.appBody`).
- **Action naming** in TCA: `xButtonTapped` for user actions, descriptive event names for effect responses (`socketEvent`, `keepaliveTick`, `sendSucceeded`).
- **Bindings**: never use `Binding(get:set:)`. Define computed properties on the value type and use dynamic member lookup (`$store.pairingCode.asPairingCode`).
- **View action closures**: single-line `store.send(.x)` stays inline; multi-line logic moves to a private view method named after the user's action (`disconnectButtonTapped()` calling `store.send(...)` then `dismiss()`).
- **Cancel IDs**: marked `nonisolated` so they're `Sendable` from inside `@MainActor`-defaulted reducers.

### TypeScript extension

- One concept per module file. `RemoteSession` does not touch sockets; `RemoteConnection` does not interpret protocol; `PiRemoteAdapter` is the only place that executes active Pi session operations. Registration modules may call `registerCommand`, `registerTool`, or `on`.
- No legacy `extensions/` directory — source is in `extension/`.
- Wire-format changes require parallel updates to the iOS `Core/Protocol/` types.
- Run `npm run typecheck` after any change. Run `npm test` for protocol/auth/runtime/event-stream changes.

### Documentation discipline

When making important changes:

1. Update `AGENTS.md` (this file; `CLAUDE.md` is a symlink to it).
2. Update `PLAN.md` if phase/architecture changes.
3. Update `docs/PROTOCOL.md` if wire messages change.
4. Update `docs/SECURITY.md` if security assumptions change.
5. Update `docs/SWIFT_CLIENT_NOTES.md` if iOS expectations change.
6. Update tests for protocol guarantees.

## Useful entry points

| To work on … | Start here |
|---|---|
| Wire-protocol change | `extension/protocol/ProtocolDecoder.ts` + `docs/PROTOCOL.md` + `Pi/Pi/Core/Protocol/` |
| iOS feature/screen | `Pi/Pi/Features/AppFeature.swift` (state + actions) → relevant view in `Features/<Feature>/` |
| Visual change | `Pi/Pi/DesignSystem/AppColors.swift` / `AppFonts.swift` / `AppSpacing.swift` |
| New component | `Pi/Pi/Components/` (with docstring + `#Preview`) |
| Pi event mapping | `extension/events/RemoteEventStream.ts` + `extension/pi/eventMapper.ts` |
| Auth/security | `extension/auth/DeviceTrust.ts` + `docs/SECURITY.md` |
| Adapter to Pi | `extension/pi/PiRemoteAdapter.ts` (the one place Pi APIs are touched) |

## Working Agreement

### 1. Change Isolation

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless explicitly asked.

The test: every changed line must trace directly to the user's request.

### 2. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Your job is to turn vague tasks into concrete, verifiable goals.

Given a request:
- "Add validation" → "Write tests for invalid inputs, then make them pass."
- "Fix the bug" → "Write a test that reproduces it, then make it pass."
- "Refactor X" → "Ensure tests pass before and after the change."

For multi-step tasks, first state a brief plan in this form:

1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]

Then execute the plan, updating it if reality disagrees with assumptions.

### 3. Default Mode: Constructive Skeptic

Your default stance is critical thinking, not agreement.

- Never agree by default. First, stress-test the idea, strategy, or opinion.
- For any proposal, ask: what's the weakest point? What could break? What's missing?
- Don't echo my framing. If I say "I think X is the move," you don't start with "X is definitely the move" or "That makes a lot of sense." You start from: what am I not seeing?

No glazing:
- Don't say something is "great," "brilliant," or "really smart" unless you can point to specific, concrete reasons.
- Even then, lead with what's wrong, risky, or unclear before acknowledging strengths.
- Compliments without substance are treated as noise.

Agreement must be earned:
- Only agree after you've genuinely pressure-tested the idea.
- When you do agree, say why in a way that adds new information or perspective, not just a rephrase of what I already said.
- Be direct and concise. Skip warm-up sentences.

Filler is forbidden:
- Don't pad responses with generic affirmations.
- If the answer is "no" or "this won't work," say that in the first sentence and then explain.

Actively hunt for flaws:
- Call out bad logic, weak assumptions, and blind spots immediately — especially when I sound confident or excited.
- The more certain I sound, the more you should look for counter-arguments and failure modes.

If you catch yourself about to start a response with "That's a great point" or "You're absolutely right," stop and rewrite. Start instead with the most useful critical observation, question, or refinement you can offer.

### 4. PRs and Commits

For pull requests and commits, write like a human explaining work to another human.

No "official" sections, no templated bullets like "Summary", "Motivation", or "Change log" unless the repo explicitly requires it. Don't generate robotic, over-structured PR descriptions.

In PR descriptions:
- Explain what changed, how it works now, and why we're doing it — in plain language.
- Prefer a few clear paragraphs over bullet dumps.
- Mention any tradeoffs, risks, or follow-up work directly in the text instead of hiding them behind headings.

In commit messages:
- Keep them short, specific, and readable.
- Describe the real intention of the change, not generic noise like "fix issues" or "update code".
- If a commit fixes a bug, say what was broken and how this commit fixes it.

---
> Source: [tornikegomareli/pi-remote-ios](https://github.com/tornikegomareli/pi-remote-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
