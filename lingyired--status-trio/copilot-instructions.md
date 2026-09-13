## status-trio

> The release workflow is the acceptance environment:

# Status Trio Agent Rules

## Highest Priority: Match the CI Toolchain

The release workflow is the acceptance environment:

- Runner: `macos-15`
- Xcode: `16.4`
- Swift: `6.1.2`

A newer local toolchain is useful, but it is not proof that CI will compile. Swift code must remain buildable with the CI toolchain.

Before committing Swift changes:

```bash
swift test
swift build -c release
```

If a change touches actor isolation, `@MainActor`, `deinit`, SwiftUI bindings, generics, or `Bundle.module` resources, also run a non-publishing release workflow before merging or publishing:

```bash
gh workflow run release.yml \
  --repo lingyired/status-trio \
  --ref <branch> \
  -f version=<next-version> \
  -f build=<next-build> \
  -f publish=false

gh run watch <run-id> --repo lingyired/status-trio --exit-status
```

Do not create a release if that preflight has not passed.

## Swift 6.1 Compatibility Rules

- Do not use `isolated deinit` or enable the `IsolatedDeinit` experimental feature. Use `deinit` with explicit cleanup; use `nonisolated(unsafe)` only for teardown-owned storage and explain why it is safe.
- Do not pass actor-isolated methods directly as function values. Use an explicit closure instead.
- Do not write `weak let`; weak reference bindings must be `var`.
- Do not assume SwiftPM `Bundle.module` resource names or directory casing match local builds. For localized resources, try the canonical and lowercase `lproj` names and load with `Bundle(path:)`.
- Do not add syntax or language features that require Swift 6.2 or newer unless the CI runner and minimum toolchain are upgraded together.
- If the Swift compiler crashes with `IRGenRequest`, `SmallVector unable to grow`, or a signal 6, reduce the code pattern that causes the crash. Do not treat it as a flaky failure and do not hide it with experimental compiler flags.

## Release Rules

- Release notes must always be written in English. This includes GitHub Release bodies, Sparkle appcast descriptions, `release_notes` workflow inputs, and release announcements.
- Release through `.github/workflows/release.yml`; do not publish manually unless the workflow is unavailable and the user explicitly asks for a manual fallback.
- Version and build numbers must be explicit and must increase the published build number.
- Confirm tests, DMG creation, Release upload, and appcast publication in the workflow result.
- The current repository has no Developer ID certificate or notarization secrets. Releases are Ad-hoc signed; document this limitation rather than claiming notarization.

See [Swift 6.1 CI compatibility](docs/swift-6.1-ci-compatibility.md) for the incident history and examples.

---
> Source: [lingyired/status-trio](https://github.com/lingyired/status-trio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
