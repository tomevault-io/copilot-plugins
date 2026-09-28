## intern

> Intern is a native macOS app written in SwiftUI and AppKit. It supports macOS 14 and later.

# Intern

Intern is a native macOS app written in SwiftUI and AppKit. It supports macOS 14 and later.

## Build configuration

Edit `project.yml`, then run `xcodegen generate`. Commit both `Intern.xcodeproj/project.pbxproj` and `Resources/Info.plist` when generation changes them.

Keep `CFBundleIconFile` in the `info.properties` section of `project.yml`. With the current project, `INFOPLIST_KEY_CFBundleIconFile` alone does not reach the generated app metadata.

Keep Debug builds under `com.devin.typesafe.jev-launcher.debug`, with the display name `Intern Dev`. Sharing the release identifier with an unsigned Debug build causes repeated macOS folder permission prompts.

Keep index builds on `LocalIndexScanner` and preserve task cancellation through the awaited call. An unlinked detached task continues file access after the launcher closes. Cancellation tests must use fake file managers, not personal folders.

## Release packaging

Run `./scripts/make-dmg.sh` to build both Apple silicon and Intel architectures. Public releases need `SIGNING_IDENTITY` and `NOTARY_PROFILE`. Keep credentials in Keychain, never in repository files.

Sign the built Release app before copying it into the DMG. This keeps the local Release copy and packaged app under the same signing identity.

Apply the volume icon after the Finder layout script. Finder's update operation can remove an existing volume icon.

Keep each `lipo -verify_arch` invocation limited to one architecture. The installed Xcode 27 tool rejects multiple architecture arguments in this operation.

Use `Intern.dmg` and `Intern.dmg.sha256` as GitHub release asset names. The README and site use the latest-release download URL.

## Verification

Follow `TESTING.md` for app tests, packaging tests, and installer checks. Run `REQUIRE_NOTARIZATION=1 sh scripts/verify-dmg.sh Intern.dmg` before publishing a notarized release.

Packaging tests use Python's standard library and run with `python3 -B -m unittest discover -s scripts -p 'test_*.py' -v`. Do not run the live API tests or destructive system actions during release verification.

---
> Source: [dabit3/intern](https://github.com/dabit3/intern) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
