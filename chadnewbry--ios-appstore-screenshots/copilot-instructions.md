## ios-appstore-screenshots

> Use these instructions when the user wants help capturing iOS screenshots or generating App Store marketing screenshots from an app project.

# Codex Instructions for `ios-appstore-screenshots`

Use these instructions when the user wants help capturing iOS screenshots or generating App Store marketing screenshots from an app project.

## Preferred workflow: take app screenshots

When the user asks to capture app screenshots, generate a Maestro flow tailored to the target iOS app instead of writing a generic flow by hand.

### Prerequisites

- A booted iOS Simulator with the app installed
- Maestro CLI installed (`brew install maestro` or https://maestro.mobile.dev)
- This repo available if the user also wants final marketing screenshots

### Step 1: discover the app structure

Read the target app source code first.

Find the bundle identifier in this order:
- `*.xcodeproj/project.pbxproj` via `PRODUCT_BUNDLE_IDENTIFIER`
- `*.plist` via `CFBundleIdentifier`
- `Project.swift` or `Package.swift`
- Ask the user only if it cannot be determined from source

Identify the main navigation model:
- SwiftUI: `TabView`, `.tabItem`, `.tab(`, `NavigationStack`, `NavigationView`, `.tag(`
- UIKit: `UITabBarController`, `UINavigationController`, storyboard tab bar scenes

Record for each important screen:
- human-readable screen name
- visible tab label or accessibility identifier
- whether setup or sample data is required

Check for blockers before capture:
- onboarding
- login
- permission dialogs
- first-launch state in `UserDefaults`

### Step 2: generate a Maestro flow

Write the flow to `maestro/capture-screenshots.yaml` in the app project.

Use this structure:

```yaml
appId: <BUNDLE_ID>
name: "Capture App Store Screenshots"
---

- launchApp:
    appId: <BUNDLE_ID>
    clearState: true

- waitForAnimationToEnd
- takeScreenshot: "01-home"
```

Use these Maestro commands when appropriate:
- `launchApp`
- `tapOn`
- `waitForAnimationToEnd`
- `takeScreenshot`
- `scrollUntilVisible`
- `swipe`
- `assertVisible`
- `inputText`
- `hideKeyboard`
- `back`

Rules:
- add `waitForAnimationToEnd` after navigation and before screenshots
- handle onboarding or permission dialogs explicitly if they exist
- use numeric screenshot names like `01-home`, `02-library`
- prefer visible tab text first; use accessibility IDs when the UI is not text-addressable

### Step 3: run Maestro

Before running:
- verify a simulator is booted with `xcrun simctl list devices | grep Booted`
- verify the app is installed

Run:

```bash
export PATH="$PATH":"$HOME/.maestro/bin"
maestro test maestro/capture-screenshots.yaml
```

If Maestro fails:
- inspect the exact missing element or timing issue
- patch the YAML
- rerun until the flow is stable

### Step 4: move screenshots into the generator inputs directory

Maestro screenshots are usually written to the current working directory for the app project.

After a successful run:
- inspect `App-Store-Screenshots/screenshot-config.json` if it exists
- read `screenshotsDirectory` when present
- default to `App-Store-Screenshots/inputs/`
- create the directory if needed
- move the captured PNGs there
- rename files or update config paths so they match

### Step 5: optionally generate final marketing screenshots

If the user wants final App Store assets:

1. Build this repo if needed:

```bash
swift build -c release
```

2. From the app project root, run the generator:

```bash
.build/release/ios-appstore-screenshots --project-dir .
```

Expected output:
- `App-Store-Screenshots/apple/en-US/iPhone 6.5/`
- `App-Store-Screenshots/apple/en-US/iPad 13/`

## Expected behavior

When helping with screenshot capture, Codex should:
- inspect the target app before proposing a flow
- generate a project-specific Maestro YAML
- run and refine the flow when possible
- move captured images into the generator input directory
- optionally run `ios-appstore-screenshots`

Do not stop at a generic YAML example if the app source is available.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chadnewbry)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chadnewbry)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
