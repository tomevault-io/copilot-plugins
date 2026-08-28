## auto-rebuild

> Auto-rebuild and reinstall VibeWindowManager on this Mac after code edits


# Auto-rebuild rule (macOS)

After any non-doc code change in this repo, rebuild and redeploy to this Mac. Skip for doc-only or `.cursor/**` changes.

This repo lives under a file-provider dir (`code.nosync`) which auto-applies `com.apple.FinderInfo` to build outputs and breaks codesign. Build to a derived-data path OUTSIDE that tree.

Run from the repo root:

```bash
DD="$HOME/Library/Developer/Xcode/DerivedData/VibeWM-auto"
xcodebuild -project VibeWindowManager.xcodeproj \
  -scheme VibeWindowManager -configuration Debug \
  -destination 'platform=macOS' \
  -derivedDataPath "$DD" build
```

On success, swap `/Applications` and relaunch:

```bash
APP="$DD/Build/Products/Debug/VibeWindowManager.app"
osascript -e 'quit app "VibeWindowManager"' 2>/dev/null || true
sleep 1
rm -rf /Applications/VibeWindowManager.app
cp -R "$APP" /Applications/
open -a VibeWindowManager
```

Rules:
- If build fails with "resource fork, Finder information, or similar detritus": confirm derivedDataPath is NOT under `code.nosync`. Never build to `./build` here.
- If build fails for other reasons, print the first real error and stop. Do not retry blindly.
- Do not open Xcode.
- Do not run this for doc-only, README, or `.cursor/**` edits.

---
> Source: [Tguntenaar/VibeWindowManager](https://github.com/Tguntenaar/VibeWindowManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
