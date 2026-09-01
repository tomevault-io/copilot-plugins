## setu-ios

> `setu_ios` is the native SwiftUI iOS client for XueLiang Cloud (雪涼云). It reimplements the existing `setu_cloud` feature set natively — it is **not** a WebView wrapper (see `docs/adr-0001-native-ios-shell.md`). It reuses the existing `setu_api_full` Spring Boot backend and its browser auth model.

# AGENTS.md

## Project Overview

`setu_ios` is the native SwiftUI iOS client for XueLiang Cloud (雪涼云). It reimplements the existing `setu_cloud` feature set natively — it is **not** a WebView wrapper (see `docs/adr-0001-native-ios-shell.md`). It reuses the existing `setu_api_full` Spring Boot backend and its browser auth model.

Feature surface: dashboard, auth (login/register/recovery/passkey), points & API keys, collections + public square, AI drawing (create/history/square/assets, Live Activities), music (search/playlists/history/background playback), gallery uploads, notifications, system status, and a full admin console.

App display language is `zh-Hans`; user-facing strings are Chinese. Keep new UI copy Chinese unless told otherwise.

## Workspace Context

This repository is usually opened inside the `setu-workspace` multi-repo workspace. Before any auth, API contract, response-shape, or cross-product-flow change, also read:

- Workspace root `AGENTS.md`
- `docs/agents/system-context.md`, `docs/agents/auth-contract.md`, `docs/agents/frontend-backend-contract.md` (in the workspace root)

The workspace path differs across machines — rely on sibling repository names (`setu_api_full`, `setu_cloud`), not absolute paths. Treat `setu_cloud` only as a feature/interface reference; do not port Vue views or its Aurora Glassmorphism styling into iOS. For backend-facing changes, inspect the matching `setu_api_full` controller, service, DTO/entity, mapper interface, and XML mapper before handoff.

## Stack

- Swift 5.9, SwiftUI, `async/await`, `URLSession`, `Codable`
- Deployment target iOS 17+ (iOS 27 capabilities are additive, availability-gated — base features must never depend on unstable APIs)
- CryptoKit for HMAC request signing; Keychain for `signSecret`; shared `HTTPCookieStorage` for `SID`
- SwiftPM (`Package.swift`) for the core library + tests; XcodeGen (`project.yml`) for the app/widget Xcode project
- WidgetKit + ActivityKit (Live Activities), APNs, AVFoundation (background audio)

## Project Structure

Two SwiftPM targets plus a widget extension and tests:

- `Sources/SetuIOSCore` — networking, auth, models, security. No UI. Layout:
  - `Core/Networking/` — `APIClient`, `AuthSigner`, and one `*Client` per domain
  - `Core/Models/` — `Codable` DTOs, one `*Models.swift` per domain
  - `Core/Security/KeychainStore.swift`, `Core/AuthSession.swift`, `Core/AppEnvironment.swift`, `Core/AppConfig.swift`, `Core/LoadState.swift`
- `Sources/SetuIOSApp` — SwiftUI app. `Features/<Domain>/` views, `Features/AppShell/` (`RootAppView`, `AppRoute`, tab/router), `Resources/Assets.xcassets`
- `Sources/SetuIOSLiveActivityWidget` — Live Activity widget extension
- `Tests/SetuIOSAppTests` — unit tests (client/DTO/auth focused), one `*ClientTests.swift` per domain

Bundle IDs: app `icu.yukiryou.setuios`, core `.core`, widget `.liveactivity`.

## Build & Test

- Fast unit check (core + clients, no simulator): `swift test`
- Regenerate the Xcode project after editing `project.yml` or adding files/targets: `xcodegen generate`
- Full app build: `xcodebuild -project SetuIOSApp.xcodeproj -scheme SetuIOSApp -destination 'platform=iOS Simulator,name=iPhone 15' build`

`Package.swift` and `project.yml` are two views of the same sources — keep target membership and paths consistent across both. Do not hand-edit `SetuIOSApp.xcodeproj`; change `project.yml` and regenerate.

## Conventions

1. **Add a domain feature bottom-up**: DTOs in `Core/Models/`, a `*Client` in `Core/Networking/`, then the SwiftUI view under `Features/<Domain>/`, then a `*ClientTests` case. Register navigation in `AppRoute.swift` / `RootAppView.swift`.
2. **Networking** goes through `APIClient`; per-domain `*Client` structs own their endpoints and decoding. Keep `SetuIOSCore` UI-free.
3. **Async state** in views uses `LoadState<T>` (idle/loading/loaded/failed) — always render loading, empty, error, and unauthorized states.
4. **Navigation**: each tab owns an independent `NavigationStack` driven by `TabRouter`/`RouterPath`; push via typed `AppRoute` cases, not ad-hoc destinations.
5. **Admin entry points** render only for `role == admin`.
6. Match surrounding code style; do not reformat unrelated files.

## Auth & Security (non-negotiable)

- Preserve the browser auth model: HttpOnly `SID` cookie + Keychain-held `signSecret`.
- Sign protected requests with HMAC over `{timestamp}:{nonce}:{METHOD}:{path}` (`X-Timestamp`, `X-Nonce`, `X-Signature`) — see `docs/backend-mobile-contract.md`. `AuthSigner` owns this; changes require matching `setu_api_full` `SignatureService` review and updated tests.
- Never store account passwords on device. Never log `SID`, `signSecret`, API keys, or push tokens.
- Keep API-key auth (`/music/**`, image API) separate from user-session auth; logged-in users use `/user/**`.
- Do not commit APNs private keys, signing assets, or provisioning profiles.

## UI & Design

- Follow the system design language, not the Web console's look. The target visual system is the soft-pink theme in `docs/ios-soft-pink-ui-design.md` — build shared `Setu*` design-system components rather than hardcoding colors per view. Global tint is `SetuColor.brandPink`.
- Support Dynamic Type, light/dark mode, and VoiceOver; keep tap targets ≥ 44pt.
- Inspect layouts at iPhone SE (375pt) and Pro Max (430pt) for overflow, clipping, and overlap.

## Verification

- Run `swift test` for any `Core` (client/DTO/auth/signing) change; add or update the matching `*ClientTests`.
- Build the app target when touching views, navigation, resources, `project.yml`, or `Package.swift`.
- Auth, HMAC signing, cookie handling, background audio (Range behavior), APNs, and Live Activities need real-device / production-domain verification — call out anything only checkable on hardware.

## Docs

- `docs/adr-0001-native-ios-shell.md` — native-shell decision
- `docs/ios-app-foundation-plan.md` — foundation plan & module order
- `docs/backend-mobile-contract.md` — `/mobile/**`, APNs, Live Activity contract
- `docs/frontend-feature-map.md` — Web feature → iOS module coverage
- `docs/ios-soft-pink-ui-design.md` — soft-pink design system & page-structure plan
- `docs/ios-product-polish-plan.md` — productization priorities, target UX, rollout phases, and acceptance criteria
- `docs/passkey-associated-domains.md` — passkey / associated domains setup

---
> Source: [yoshino-xiao7/setu_ios](https://github.com/yoshino-xiao7/setu_ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
