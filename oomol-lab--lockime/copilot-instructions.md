## lockime

> Design spec: `docs/DESIGN.md`. Release process: `docs/RELEASING.md`.

# LockIME

Design spec: `docs/DESIGN.md`. Release process: `docs/RELEASING.md`.
Build/test: `make build` / `make test` (xcodegen + xcodebuild; see `Makefile`).
Update flows: `make update-test-{none,download-fail,extract-fail,success}`
runs Sparkle against a local feed (see `scripts/update-lab/README.md`).

## Localization (i18n) — hard rules

The app has an **in-app language override** (`LanguagePreference`), so the
macOS system language is irrelevant to what the user must see. Consequences:

- **Never display text localized by someone else's bundle.** Foundation/Sparkle
  `error.localizedDescription` resolves against the *system* language and
  produces mixed-language UI. Map errors to semantic categories whose messages
  are catalog keys (see `UpdateFailure`), and log the original error instead.
- SwiftUI surfaces resolve string literals live via the injected `\.locale`.
  AppKit surfaces (`NSAlert`, window titles) bypass that — route them through
  `AppKitStrings` / `AppState.loc`. This includes SwiftUI modifiers that
  *bridge into AppKit*: `.navigationTitle("Key")` resolves the key against the
  system language when it becomes the `NSWindow` title — pass
  `state.loc("Key")` instead.
- Third-party **views** are third-party bundles too: KeyboardShortcuts'
  `Recorder` localizes its placeholder and conflict alerts from its own
  `.lproj`s against the system language. Any package that draws its own text
  must have its resource bundle listed in `ThirdPartyBundleLocalization`,
  which re-classes it to resolve in the app's chosen language (English
  fallback, never system).
- Every user-facing string needs an entry in `Localizable.xcstrings` translated
  for **all** `SupportedLanguage` cases. Keys reached only via
  `loc(...)`/`LocalizedStringKey(variable)` are invisible to Xcode's extractor —
  add them to the catalog by hand.
- `Tests/LockIMEKitTests/LocalizationGuardTests.swift` enforces the above
  (no `localizedDescription` in `Sources/LockIME`, no literal
  `.navigationTitle("...")`, redirected third-party bundles exist and cover
  every `SupportedLanguage`, full catalog coverage, dynamic keys exist). Keep
  it green; extend it when adding new dynamic-key entry points or third-party
  UI packages.
- Manual smoke test for any new surface: set the app language to something
  *different* from the system language and walk the flow — any mixed-language
  screen is a bug.

## Documentation translations — hard rules

`README.md` ships in **English + Simplified Chinese only**. English is the
authoritative source; `README.zh-CN.md` is its translation, kept in sync. The
docs under `docs/` (`DESIGN.md`, `RELEASING.md`) are **English-only — do not
translate them.**

- **Naming:** English is the base name (`README.md`); the Chinese translation is
  `README.zh-CN.md` (region code `zh-CN`, *not* the script code `zh-Hans`).
- **Language switcher:** both READMEs carry a nav line directly under the H1 —
  `**English** · [简体中文](README.zh-CN.md)` and
  `[English](README.md) · **简体中文**` — with the current language **bold and
  unlinked**. The autonyms `English` / `简体中文` are never translated.
- **Intra-doc links point at the English docs.** `README.zh-CN.md` links to
  `docs/DESIGN.md` / `docs/RELEASING.md` (there are no translated docs).
- **Never translate code.** Fenced/inline code, shell commands, file paths,
  identifiers, `make` targets, env-var and secret names, URLs, hex colors,
  version strings, framework and brand names (LockIME, Sparkle, SwiftUI,
  macOS, Tahoe, …) stay byte-for-byte identical. The H1 keeps the bare
  `# LockIME` brand.
- **When you edit `README.md`, update `README.zh-CN.md` in the same change**
  (or explicitly flag the drift).

---
> Source: [oomol-lab/LockIME](https://github.com/oomol-lab/LockIME) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
