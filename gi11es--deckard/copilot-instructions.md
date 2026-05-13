## deckard

> xcodebuild -project Deckard.xcodeproj -scheme Deckard -configuration Debug build

# Deckard

## Build & Run

```bash
# Build
xcodebuild -project Deckard.xcodeproj -scheme Deckard -configuration Debug build

# App location
/Users/gilles/Library/Developer/Xcode/DerivedData/Deckard-hkgvzqxyznptcubawtorcnhugxer/Build/Products/Debug/Deckard.app

# Quit and relaunch (osascript is required — pkill does not work for this app)
osascript -e 'tell application "Deckard" to quit'
open /Users/gilles/Library/Developer/Xcode/DerivedData/Deckard-hkgvzqxyznptcubawtorcnhugxer/Build/Products/Debug/Deckard.app
```

**Always ask for confirmation before restarting Deckard** — do not quit and relaunch the app automatically after a build. Ask the user first.

## Releasing a New Version

Releases are automated via [release-please](https://github.com/googleapis/release-please). Use **conventional commits** (`feat:`, `fix:`, `chore:`, etc.) on `master`.

### How it works

1. Commits land on `master` with conventional commit messages.
2. release-please opens (or updates) a release PR that bumps the version and updates `CHANGELOG.md`.
3. Merging that PR creates a `vX.Y.Z` tag, which triggers the build job to build, sign, and publish the DMG.

### Version locations (managed by release-please)

These files contain `x-release-please-version` annotations and are bumped automatically:

1. `Resources/Info.plist` — `CFBundleShortVersionString`
2. `Sources/Window/SettingsWindow.swift` — version label in About pane
3. `README.md` — download badge version (uses shields.io query-param format to avoid conflicts with the generic replacer)

Do not bump versions manually — merge the release-please PR instead.

## Screenshot

The README screenshot (`screenshot.png`) uses transparent corners. To update it:

```bash
# Trim 1px border, apply 12px rounded-corner mask, save as PNG32 with alpha
magick screenshot_raw.jpg -crop {W-2}x{H-2}+1+1 +repage -alpha set \
  \( -size {W-2}x{H-2} xc:none -fill white -draw "roundrectangle 0,0 {W-3},{H-3} 12,12" \) \
  -compose DstIn -composite PNG32:screenshot.png
```

Must be PNG (JPG doesn't support transparency). Don't use flood fill — JPEG compression causes it to bleed into dark window content.

## Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/) prefixes:

- `fix:` — bug fix
- `feat:` — new feature
- `refactor:` — code change that neither fixes a bug nor adds a feature
- `chore:` — build, CI, dependencies, tooling
- `docs:` — documentation only

Example: `fix: use length-bounded reads for terminal URL actions (#6)`

---
> Source: [gi11es/deckard](https://github.com/gi11es/deckard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
