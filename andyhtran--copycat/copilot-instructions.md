## copycat

> **CLI-only** — no Xcode. All builds use `swift build` via SPM. Use `just` as the task runner.

# CopyCat

**CLI-only** — no Xcode. All builds use `swift build` via SPM. Use `just` as the task runner.

- macOS 14.0+, Swift 6.2+ (hot key teardown uses `isolated deinit`, non-experimental only from 6.2)

```bash
just dev          # Kill, build, sign, install to /Applications/CopyCat Dev.app, launch
just clean        # Remove build artifacts
just reset-grants # Wipe Accessibility grants
```

## Releasing

Bump `MARKETING_VERSION` in `version.env`, push to main, then `just publish` — signs, notarizes, creates a GitHub release, and updates the Homebrew tap.

## Icons

Canonical sources are `Icons/AppIcon.svg` and `Icons/MenuBarIcon.svg`. Regenerate `Resources/AppIcon.icns` and `Resources/MenuBarIcon.pdf` via `bash Scripts/build-icons.sh`.

`MenuBarIcon.svg` ships as a template image (solid black on transparent only) — don't add fills, gradients, or non-alpha effects.

---
> Source: [andyhtran/CopyCat](https://github.com/andyhtran/CopyCat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
