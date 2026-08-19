## herdrm

> Native macOS console for [herdr](https://herdr.dev) (the terminal workspace manager for

# herdrm — HerdrM

Native macOS console for [herdr](https://herdr.dev) (the terminal workspace manager for
coding agents). Sidebar lists Spaces (herdr workspaces) and Agents; the bottom-left
footer switches devices (Local + remote herdr hosts over SSH); the right pane is a pure
embedded terminal running `herdr agent attach` — no chat composer.

Design canvas (waku-style sidebar, light/dark): `design/` — published as the
"Herdr for Mac" artifact. `design/canvas.json` notes carry the design tokens and specs.

## Layout

- `Packages/HerdrKit` — SPM library: NDJSON-over-Unix-socket RPC (`SocketRPC`),
  models, `SSHTunnel` (OpenSSH `-L local.sock:remote.sock` forward), `Device`/`DeviceStore`
  (persisted to `~/Library/Application Support/HerdrM/devices.json`), `HerdrService` facade.
- `Sources/HerdrM` — SwiftUI app (XcodeGen `project.yml`), SwiftTerm terminal embed.
- `design/` — design canvas working files (`*.dc.html` artboards + `canvas.json`).

## Build & test

```sh
make build      # xcodegen + xcodebuild → build/Build/Products/Debug/HerdrM.app
make run
make kit-test   # HerdrKit integration tests (need a running local herdr)
HERDRM_E2E_SSH_TARGET=vincent@10.10.10.87 make kit-test   # + remote SSH E2E
```

xcodebuild needs `-skipPackagePluginValidation` (SwiftTerm ships a build plugin);
the Makefile passes it.

## Release

Repo: github.com/missuo/herdrm. Push a `v*` tag → `.github/workflows/release.yml`
builds Release (Developer ID: MOE AI LLC, hardened runtime), notarizes via
notarytool, staples, Sparkle-signs the zip, generates `appcast.xml`, and
publishes both as a GitHub release. Secrets: MACOS_CERTIFICATE_P12/_PASSWORD,
APPLE_ID, APPLE_TEAM_ID, APPLE_APP_PASSWORD, SPARKLE_PRIVATE_KEY (EdDSA private
key also lives in the local login Keychain; public key is pinned in project.yml).
Sparkle feed: the release asset `appcast.xml` at `releases/latest/download/`.
Versioning: MARKETING_VERSION from the tag, CFBundleVersion = CI run number.
CHANGELOG.md is mandatory: CI extracts the `## [x.y.z]` section for the GitHub
release notes and the Sparkle update description, and fails if it's missing —
add the section before tagging. The cask in OwO-Network/homebrew-brew is
auto-bumped after each release.

## herdr protocol notes (0.8.0, protocol 19; verified against the live socket)

- Requests are NDJSON `{"id","method","params"}` on `~/.config/herdr/herdr.sock`;
  `params` must be present even when empty (`{}`), or the server rejects the request.
- `tab.create` returns the new pane as `result.root_pane.pane_id`.
- `events.subscribe` takes `{"subscriptions":[{"type":"pane.updated"},…]}`;
  `pane.agent_status_changed` / `pane.scroll_changed` / `pane.output_matched` are
  pane-scoped (require `pane_id`) and cannot be subscribed globally — status changes
  arrive globally as `pane.updated`. Full global kind list: `HerdrEvent.allKinds`.
- Terminal attach: `herdr agent attach <pane_id> --takeover` (takes the pane over from
  other attached clients). Remote devices run it through `ssh -tt` with PATH prepended
  (`sshd` exec is not a login shell; herdr lives in `/opt/homebrew/bin` on macOS hosts).
- Agent status buckets sort Blocked > Done > Working > Idle (matches Heeler).

Reference repos: `~/Projects/herdr` (server source), `~/Projects/Heeler` (iOS client,
same domain model), `~/Projects/waku` (sidebar design reference).

---
> Source: [missuo/herdrm](https://github.com/missuo/herdrm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
