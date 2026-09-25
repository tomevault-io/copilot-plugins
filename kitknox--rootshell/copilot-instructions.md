## rootshell

> Conventions in this repo that are not obvious from the code. See

# Notes for coding agents

Conventions in this repo that are not obvious from the code. See
[README.md](README.md) for what rootshell is and how to build it.

## Signing identity is centralized — never hardcode it

`Configuration/Identity.xcconfig` is the single source of truth for the
Apple Developer team, the bundle-identifier prefix, the App Groups, the
keychain group, and the iCloud container. No target sets `DEVELOPMENT_TEAM`
itself.

To build under a different Apple Developer account, run:

```bash
scripts/setup-dev-signing.sh
```

It writes the gitignored `Configuration/DeveloperSettings.xcconfig`, which
`Identity.xcconfig` includes optionally. Read
[docs/contributor-signing.md](docs/contributor-signing.md) before changing
anything about signing.

**Do not** edit `project.pbxproj`, an `.entitlements` file, or an `Info.plist`
to change the team or bundle identifier. Those files reference variables
(`$(ROOTSHELL_DEVELOPMENT_TEAM)`, `$(ROOTSHELL_DEFAULT_APP_GROUP)`, and
friends); replacing one with a literal breaks every other configuration
silently, because a target-level build setting shadows the whole xcconfig
chain.

Code that needs a resolved identifier at runtime reads it back from the
bundle's Info.plist rather than repeating the literal:
`rootshell/Core/Security/AppIdentifiers.swift` for the app group, keychain
group, and iCloud container; `rootshell-helper/Sources/SocketCommandServer.swift`
for the bundle identifier and team; `RootshellPushKit`'s `PushConfiguration`
for its app group.

## Check project.pbxproj before committing

`git diff` on `project.pbxproj` should be empty unless you meant to change the
project. A signing override resolves through variables, so a build under your
own team shouldn't touch it — if a hunk appears anyway, it likely carries your
org identifier or team, and shouldn't go into a PR.

## Building

Shared schemes: `rootshell-AppStore` (iOS/iPadOS/visionOS, sandboxed),
`rootshell-Standalone` (unsandboxed Mac Catalyst), `rootshell-China`,
`rootshell-helper` (the macOS background helper), and `rootshellvpn`
(the native macOS VPN host and system extension).

Build for a simulator by device id rather than
`-destination 'generic/platform=iOS Simulator'`; the generic destination builds
x86_64 too and fails with `cannot find type 'arm_thread_state64_t' in scope` in
`MainThreadStackSampler.swift`.

```bash
xcodebuild -project rootshell.xcodeproj -scheme rootshell-AppStore \
  -destination 'id=<simulator-udid>' build
```

Device and Mac Catalyst builds need `-allowProvisioningUpdates` from the
command line; without it every target fails with "Automatic signing is disabled
and unable to generate a profile". They also need an Apple ID signed in under
Xcode → Settings → Accounts. Note that a `No Accounts` error line is not
diagnostic on its own — Xcode emits it alongside unrelated capability failures
even with an account signed in and a valid team profile in use.

`rootshellvpn` has a shared native macOS scheme and embeds the `tunnel` system
extension. Both targets explicitly clear the terminal app's bridging header.
`rootshell-Standalone` bundles the prebuilt app in `rootshell/Resources`; it
does not rebuild these targets. Use `scripts/deploy-vpn-host.sh --build-only`
for an unsigned universal compile check, or the signed workflow in
[docs/contributor-signing.md](docs/contributor-signing.md) to refresh that
artifact. The latter requires Developer ID profiles and notarization.

To compare resolved build settings across configurations, pass `-alltargets`
rather than looping over targets — one invocation per configuration instead of
one per target.

---
> Source: [kitknox/rootshell](https://github.com/kitknox/rootshell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
