## kiri-friends

> Use these documents as the project map before making architectural, runtime, UI, or workflow changes:

# Kiri Friends Agent Guidelines

## Development Documentation

Use these documents as the project map before making architectural, runtime, UI, or workflow changes:

- `docs/README.md` — documentation index.
- `docs/product/information-architecture.md` — product scope and information architecture for watchOS.
- `docs/interfaces/watchos-swiftui-interface.md` — watchOS SwiftUI interface structure, complications, and navigation.
- `docs/interfaces/mac-buddy.md` — SwiftUI macOS Mac Buddy (CLI host bridge + desktop pet, AGPL-3.0).
- `docs/core/cli-bridge.md` — CLIBridge protocol between iPhone companion and CLI tools.
- `docs/core/cli-plugins.md` — plugin/hook contracts for the twelve supported CLI hosts.
- `docs/core/cli-adapters.md` — normalized CLI adapter behavior.
- `docs/server/relay-server.md` — Cloud Relay server routing, downlink delivery, presence, and retries.
- `docs/core/security-and-privacy.md` — trust boundaries, redaction, pairing, and approval safety.
- `docs/operations/app-store-distribution.md` — TestFlight and App Store submission process.
- `docs/operations/license-boundaries.md` — MIT / AGPL-3.0 split between the watch + iPhone stack and the Mac Buddy.
- `docs/quality/testing-quality.md` — testing and quality expectations.

watchOS-specific implementation guidance lives under `.agents/skills/`, especially:

- `.agents/skills/watchos-design-guidelines/SKILL.md`
- `.agents/skills/watchOS/SKILL.md`
- `.agents/skills/watchos-code-review/SKILL.md`
- `.agents/skills/kiri-friends-core/SKILL.md`

## Documentation Maintenance

When a change meaningfully alters product behavior, architecture, runtime configuration, communication protocol, testing expectations, or UI information architecture, update the relevant document in `docs/` in the same change set. Do not let implementation and documentation drift.

## UI Copy Constraints

- Avoid redundant copy. Do not repeat information already expressed by a title, metric, selected state, icon, or surrounding section.
- Prefer concise labels over explanatory text when the UI state is self-evident.
- Remove disabled placeholder actions unless they teach a real next step.
- Do not add low-information detail text such as "Current status", "selected", or repeated counts when nearby UI already communicates the same fact.
- Keep user-visible copy in English unless explicitly asked otherwise.

## watchOS Native Component Constraints

- Prefer native SwiftUI and watchOS controls (`List`, `Form`, `Button`, `Picker`, `Toggle`, `TabView`, `NavigationStack`) before custom components.
- Do not recreate native selection, navigation, or button behavior with overlays, fake masks, or hand-rolled hit targets.
- Prefer native watchOS APIs for complications (`CLKComplicationWidget`, `WidgetKit`) and notifications (`WKUserNotificationInterfaceController`).
- Keep custom views small and compositional. Extract only when it clarifies state, layout, or reuse.
- Respect the small screen size of Apple Watch. Prioritize glanceable information over dense content.

## Multi-Agent State Pipeline

- The iPhone companion is the only client that talks to Cloud Relay. The watchOS app and the WidgetKit complication consume state through it.
- `apps/apple/Sources/KiriFriendsBridge/BridgeRuntime.swift` owns the pipeline: pulls `/v1/events`, folds into `BridgeStateStore`, syncs Watch via `PhoneWatchConnectivityController`, mirrors to `AppGroupSnapshotStore`, and forwards `WatchAction`s back as relay requests.
- WatchConnectivity `applicationContext` is a multi-kind envelope (`state.snapshot` / `buddy.settings`); never overwrite the whole context, always merge through `WatchConnectivityEnvelope.merge`.
- `StateSnapshot.session` is the priority-resolved primary session; `StateSnapshot.sessions` carries the full list. Always treat the array as the authoritative multi-agent view and the single `session` as a glanceable shortcut.
- `CLITool` enum now covers all twelve CLI hosts. Unknown raw values decode to `.unknown` instead of failing, so older watch builds still display newer agents.
- The Cloud Relay HTTP layer lives at `server/src/http-server.ts`; bring it up with `make dev-relay` before exercising the iPhone↔Watch path locally.

## Apple Surfaces AGPL Boundary

- Every Apple-platform binary is AGPL-3.0: `KiriFriendsWatchApp`, `KiriFriendsWatchKit`, `KiriFriendsWidgets`, `KiriFriendsPhoneApp`, `KiriFriendsBridge`, `KiriFriendsBuddyMac`, `KiriFriendsMacBuddyKit`, plus the Mac Buddy test target. They all carry assets and/or runtime semantics from [clawd-on-desk](https://github.com/rullerzhou-afk/clawd-on-desk).
- MIT territory is limited to: `KiriFriendsCore`, `KiriFriendsCLI`, `apps/apple/Tests/KiriFriendsCoreTests/`, `server/`, `plugins/`, `fixtures/`, `docs/`.
- Dependencies must point AGPL → MIT, never the reverse. AGPL targets may import `KiriFriendsCore`; nothing in `KiriFriendsCore` may import an AGPL target.
- New shared types belong in `KiriFriendsCore`. Watch/iPhone/Widget/Bridge-specific code belongs inside the corresponding AGPL directory.
- Distribution is **self-build and internal TestFlight only**; public App Store / external TestFlight are intentionally out of scope. See [docs/operations/license-boundaries.md](docs/operations/license-boundaries.md) and [docs/operations/app-store-distribution.md](docs/operations/app-store-distribution.md).

---
> Source: [Project-Kiri/Kiri-Friends](https://github.com/Project-Kiri/Kiri-Friends) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
