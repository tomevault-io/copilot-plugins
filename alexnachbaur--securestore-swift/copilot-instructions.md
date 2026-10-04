## securestore-swift

> Instructions for AI agents working in the securestore-swift repository itself. If you are

# AGENTS.md

Instructions for AI agents working in the securestore-swift repository itself. If you are
*integrating* SecureStore into another project, read [llms.txt](llms.txt) and the
[README](README.md) instead.

## What this package is

A cross-platform secure-credential store: one `SecureStore` protocol (set, read, remove,
removeAll, `keys(withPrefix:)`) and four backends, selected at compile time — Keychain Services
on Apple, Credential Manager on Windows, the freedesktop.org Secret Service on Linux, and a
host-registered C bridge for platforms with no Swift-reachable store (Android). Swift 6.3+, one
library target plus the `CSecret` system-library shim.

The only external dependency is libsecret, and it is scoped to Linux by a
`.when(platforms: [.linux])` condition on the target dependency — that condition is load-bearing,
because without it `libsecret-1-dev` becomes a build requirement on every platform. This package
is linked into credential paths on multiple platforms; do not add another dependency, or widen
that condition, without raising it first.

## Non-negotiable design rules

These are properties, not preferences. A change that breaks one is wrong even if it compiles and
passes.

1. **No Apple concepts in the cross-platform API.** `namespace` is not `accessGroup`. If you
   cannot express something without a platform term, it belongs in a backend.
2. **Reads throw.** A missing item is `nil`; anything else throws. Never collapse a failure into
   `nil` — a locked keychain must not look like a signed-out user.
3. **No heap pointers cross the C boundary.** Results come back through sink callbacks that copy
   within the call's lifetime. Never return a malloc'd buffer for the other side to free.
4. **Every C operation returns a status.** `0` ok, `1` not found, anything else a host error
   surfaced verbatim in `SecureStoreError.platform`. A host may additionally register a
   describer (`securestore_register_host_describer`) to turn its codes into text.
4a. **Failures carry context, not just a number.** `SecureStoreError.platform` wraps a
   `PlatformFailure` (backend, operation, code, message, domain). Every backend resolves the
   platform's own message; a new one that does not is incomplete.
5. **Callbacks are non-capturing `@convention(c)`.** Per-call state travels through the explicit
   `context` pointer. This is what makes them safe from JNI.
6. **The C ABI is a compatibility surface.** Additive callbacks are fine; changing an existing
   signature breaks every host compiled against it and is a semver-major change.
7. **Never `try?` or `try!`.** Not in production, not in tests, not in `defer`. It discards the
   error, which is the failure-swallowing this package exists to prevent. `defer` cannot throw, so
   cleanup records a test issue instead of discarding — see `cleanUp` in the contract suite.

## Where behaviour is tested

`Tests/SecureStoreTests/SecureStoreContractTests.swift` is **not** platform-guarded, on purpose:
it runs against the real Keychain on Apple and against a host-registered in-memory backend
elsewhere, so the two cannot drift. Put behavioural assertions there, not in a backend-specific
file.

`HostBackendFixture.swift` registers that in-memory backend through the *real* `@_cdecl` entry
point, so the suite exercises the actual ABI rather than a Swift stand-in. Keep it that way — a
fixture that bypasses the C boundary would test nothing that matters.

Beware platform divergence in the APIs underneath: `SecItemDelete` removes every matching item on
iOS but exactly one on the macOS legacy keychain, which is why `removeAll()` loops. Assume other
such differences exist and prefer behaviour asserted by the contract suite over what the
documentation implies.

## Before you finish

```bash
swift format --in-place --recursive Sources Tests Package.swift
swift format lint --strict --recursive Sources Tests Package.swift
swift test
swift build --swift-sdk aarch64-unknown-linux-android28   # if the SDK is installed
```

Update [CHANGELOG.md](CHANGELOG.md) under `[Unreleased]` for anything user-visible, and say *why*
a change was made — the existing entries are written to be read a year later by someone deciding
whether a behaviour is intentional.

---
> Source: [AlexNachbaur/securestore-swift](https://github.com/AlexNachbaur/securestore-swift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
