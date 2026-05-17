## chatgpt-auth-kit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & test

Swift Package Manager, `swift-tools-version: 5.9`, iOS 16+. The package has UIKit/SwiftUI/SafariServices imports, so plain `swift build` from the host Mac fails — build & test against an iOS Simulator destination via `xcodebuild`. The test action only runs against the auto-generated `*-Package` scheme:

```sh
SIM_ID=<simulator-udid>   # from `xcrun simctl list devices` or `xcodebuild -showdestinations -scheme ChatGPTAuthKit`
xcodebuild -scheme ChatGPTAuthKit         -destination "platform=iOS Simulator,id=$SIM_ID" build
xcodebuild -scheme ChatGPTAuthKit-Package -destination "platform=iOS Simulator,id=$SIM_ID" test
xcodebuild -scheme ChatGPTAuthKit-Package -destination "platform=iOS Simulator,id=$SIM_ID" test -only-testing:ChatGPTAuthKitTests/ChatGPTAuthKitTests/testPKCEVerifierAndChallengeShape
```

`-scheme ChatGPTAuthKit` is library-only and rejects the `test` action with `Scheme … is not currently configured for the test action.` — use the `-Package` scheme for tests. `name=iPhone 15` style destinations sometimes fail to resolve when multiple installed runtimes match; pin to an `id=…` from `-showdestinations`.

## Library scope (read this before adding anything)

**Single product, zero third-party dependencies, ever.** The library does:

1. ChatGPT/Codex PKCE OAuth (`OAuthFlow` → `OAuthClient` + `LoopbackServer`).
2. Keychain-backed credential persistence (`CredentialsStore`).
3. Auto-refresh provider (`RefreshingCredentialsProvider`).
4. Three minimal hand-rolled REST/SSE clients against `chatgpt.com/backend-api/...` (`ResponsesClient`, `ModelsClient`, `UsageClient`).
5. Two SwiftUI bits (`SignInWithOpenAIButton`, `SafariView`).

Things that **do not belong** in the library, even when they look convenient:

- Any third-party SDK dependency (MacPaw/OpenAI, etc.). Plugging into other SDKs is documented in README as config the consumer adds in their own app — never as a target/product here.
- Higher-level abstractions over the Responses API (agent loops, tool-call dispatchers, conversation state).
- Anything that imports a third-party type into a public API.

If a feature feels like it belongs "on top of" what's listed above, it goes in the consumer app, not here. The README's "Plugging in other OpenAI SDKs" section is the place for integration recipes.

## Architecture

This package re-implements the Codex CLI's PKCE OAuth flow inside an iOS app, then talks to the same private Codex backend. The non-obvious design decisions:

**OAuth flow (`OAuthFlow` → `OAuthClient` + `LoopbackServer`)** — `OAuthFlow.run` is the orchestrator: build authorize URL with PKCE → start a localhost listener on port 1455 → call back into the host app's `present(URL)` closure to surface a Safari sheet → wait for the redirect → exchange code for tokens. The constants in `OAuthClient` (`clientID`, `redirectURI`, `scopes`, the extra `codex_cli_simplified_flow` and `originator` query params) are taken from the Codex CLI; changing them breaks the flow on OpenAI's auth server. `OAuthFlow` awaits `LoopbackServer.start()` (which only resolves on `NWListener.State.ready`) before calling `present(url)` — there is no "magic sleep," so don't add one back if you see a "didn't open" report.

**Why `SFSafariViewController` and not `ASWebAuthenticationSession`** — the redirect URI is `http://localhost:1455/auth/callback`. `ASWebAuth` requires a custom URL scheme and cannot intercept an `http://` callback, so we use plain Safari + a real loopback HTTP server. This is also why the host app needs `NSAllowsLocalNetworking` + `NSLocalNetworkUsageDescription` in Info.plist (see README).

**`LoopbackServer` is an actor** — minimal one-shot HTTP/1.1 listener built on `Network.framework`. Tries dual-stack first; if construction fails, falls back to separate IPv4 and IPv6 listeners (because `localhost` may resolve to `::1`). State (`listeners`, two continuations, `pendingResult`, `didFinish`, `anyReady`) is actor-isolated; `NWListener` callbacks fire on a private serial dispatch queue and hop into the actor via `Task`. Callback after-`.ready` failures of one listener are ignored (the other may still be serving). Sends the success HTML, then performs a TCP half-close (FIN) before `cancel()` so the browser actually receives the body — don't simplify this without preserving the FIN/cancel ordering.

**Codex Responses API shape (`ResponsesClient`)** — calls hit `https://chatgpt.com/backend-api/codex/responses`, *not* the public OpenAI API. Required headers: `Authorization: Bearer <accessToken>`, `ChatGPT-Account-ID: <accountID>`, `OpenAI-Beta: responses=experimental`, `originator: codex_cli_rs`. Required body fields (encoded via the `RequestBody` struct in `ResponsesClient.swift`): `instructions`, `reasoning: {effort, summary}`, `include: ["reasoning.encrypted_content"]`, `store: false`, and a structured `input` array (system collapses into `instructions`; user/assistant become `{type: "message", role, content: [{type: "input_text"|"output_text", text}]}`). A bare-string `input` or missing `instructions`/`reasoning`/`include` is rejected with a 400. Same constraints apply when consumers wire in MacPaw/OpenAI from README — that's why the README spells them out.

**Credentials flow through a `CredentialsProvider`** — `ResponsesClient` / `ModelsClient` / `UsageClient` each take a `CredentialsProvider` (`provider.currentCredentials()` is awaited per request). `RefreshingCredentialsProvider` is the production impl: it caches a `Credentials`, refreshes via `OAuthClient.refresh(...)` when `Credentials.isExpired(buffer:)` is true, coalesces concurrent refreshes through a single `Task`, and persists the new value via an optional `CredentialsStore`. The `init(credentials:)` overloads on each client wrap the value in a `StaticCredentialsProvider` (no refresh). Don't store a bare `Credentials` in long-lived state — it'll go stale and you'll start hitting 401s.

**`accountID` lives inside the JWT** — `JWT.extractAccountID` decodes the access-token payload to find `chatgpt_account_id` (under `https://api.openai.com/auth`, top-level, or as the first `organizations[].id`). Same for `Credentials.planType` via `chatgpt_plan_type`. Don't hit `/me` — it isn't called and the JWT is the source of truth. The decoder uses `Codable`, not `JSONSerialization`; we don't verify the signature because the JWT comes over TLS from the OpenAI token endpoint and is only inspected for UX hints.

**`CredentialsStore`** — single Keychain item (`kSecClassGenericPassword`, default service `ChatGPTAuthKit` / account `chatgpt-oauth`, accessibility `kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly`, optional `accessGroup` for app-extension sharing). `save` does update-then-add to handle both first-run and refresh. The struct is `@unchecked Sendable` because its only "non-Sendable" field is a `CFString` accessibility constant, which is always one of the immutable `kSecAttrAccessible…` globals.

**`ModelsClient` / `UsageClient`** — separate REST GETs that share the same auth header pattern (`Authorization`, `ChatGPT-Account-ID`, `originator`) but go to different endpoints (`/backend-api/codex/models`, `/backend-api/wham/usage`). All response parsing is `Codable` with a private `Payload`/`RawModel` DTO that maps snake_case JSON to camelCase Swift; `UsageClient.parse(_: Data)` is `internal` and exercised by tests directly.

**Resources** — `Sources/ChatGPTAuthKit/Resources/Media.xcassets` ships the OpenAI logo (simple-icons, MIT) used by `SignInWithOpenAIButton`; `Package.swift` declares `.process("Resources")` and the view loads it via `Image("OpenAILogo", bundle: .module)`.

## Demo app

`Demo/` is gitignored — it's a runnable iOS host that consumes the package via a local SPM reference (`relativePath = ".."`). It depends on `MacPaw/OpenAI` directly (not via the package) to demonstrate the README's "Plugging in other OpenAI SDKs" pattern, including a multi-turn tool-calling chat. The demo is the canonical "this is what your app would look like" reference; if you change the public API in a way that breaks it, fix the demo too even though it doesn't ship.

---
> Source: [MobAI-App/chatgpt-auth-kit](https://github.com/MobAI-App/chatgpt-auth-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
