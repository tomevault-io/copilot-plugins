## hermesses

> Hermesses is a Hermes-only Rokid Glasses client. Its supported backend is `hermes-webui`; do not add the removed OpenClaw protocol, a generic provider abstraction, or a direct Hermes Runs API fallback.

# Repository guidance

Hermesses is a Hermes-only Rokid Glasses client. Its supported backend is `hermes-webui`; do not add the removed OpenClaw protocol, a generic provider abstraction, or a direct Hermes Runs API fallback.

No deployment is preconfigured. The user supplies an exact HTTPS `hermes-webui` origin in the phone app.

```text
Hermes Agent <-> hermes-webui <-> phone-app <-> Rokid CXR/debug WebSocket <-> glasses-app
```

## Modules

- `phone-app/` contains the Android companion, Hermes client, Keystore credential envelope, voice, uploads, TTS, wake coordination, diagnostics, and CXR-M bridge.
- `glasses-app/` contains the Rokid HUD, gestures, camera, staged input, sessions, compact tool progress, and approval overlay.
- `shared/` contains only the backend-neutral phone-to-glasses JSON protocol.
- `contract-test/` contains pure-JVM transport tests and the optional live `hermes-webui` probe.

Packages and installed identities are `com.hermesses.*`.

## Verification

Use JDK 17 and Android SDK 34.

```bash
./scripts/check.sh
./scripts/build-release.sh
```

The first command runs offline checks and debug assemblies. The second creates signed local phone and glasses APKs and verifies that the phone embeds the exact signed glasses artifact. The live contract probe is intentionally separate:

```bash
./gradlew :contract-test:run
```

It reads `hermes.webui.url` and optional `hermes.webui.password` from gitignored `local.properties`. Never log, fixture, or commit credentials.

## Product invariants

- Voice input is explicit: long-press, speak, stage, then Send.
- Photo capture is explicit and attaches at most four images to the next turn.
- Approvals block normal HUD gestures; Deny is the safe default and Allow once requires a safe summary.
- Raw tool data, reasoning, credentials, images, server frames, user content, and exception details never cross to the HUD or diagnostic log.
- Native Hermes requests omit `Origin` and `Referer`, reject redirects and cleartext HTTP, and stay within the existing response/event/photo limits.
- Hermes DTOs stay in `phone-app`; `shared` remains backend-neutral.

## Rokid boundary

- Phone SDK: `com.rokid.cxr:client-m:1.0.8`
- Glasses SDK: `com.rokid.cxr:cxr-service-bridge:1.0`
- Local reference snapshots: `docs/rokid-sdk/` and `docs/rokid-sdk-glasses/`
- Debug bridge: phone port `8081`, glasses emulator host `10.0.2.2`

Display power is phone-owned. A neutral HUD double-tap requests native screen-off and then brightness `0` as a firmware compatibility fallback. Zero is never persisted; wake restores the last desired positive brightness. Do not substitute Android power APIs, Activity termination, or timeout changes.

Rokid credentials come only from gitignored `local.properties`:

```properties
rokid.clientSecret=...
rokid.accessKey=...
```

Keep publication separate from implementation. Do not push, publish, or deploy unless the user explicitly asks for it.

---
> Source: [zmactep/hermesses](https://github.com/zmactep/hermesses) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
