## ios-simulator-skill

> Control iOS Simulator and run Flutter iOS apps for testing and development. Use when the user wants to test a Flutter app, take a screenshot of the simulator, tap or swipe on screen, type text, check app logs, install or launch an app, run flutter build/test/run, check accessibility, or interact with iOS Simulator in any way. Also use when user mentions "simulator", "iPhone", "device", "截圖", "模擬器", or "flutter run".


# iOS Simulator Skill

Control iOS Simulator and automate Flutter iOS app testing using shell scripts.

## Setup

All scripts live in `scripts/` relative to this skill folder. Run `health_check.sh` first:

```bash
bash /path/to/ios-simulator-skill/scripts/health_check.sh
```

**Optional dependencies:**
- `idb` — required for tap, swipe, screen_map: `brew install idb-companion && pip install fb-idb`
- `Pillow` — required for visual_diff: `pip install Pillow`

## Core Principle: Use screen_map before screenshot

`screen_map.sh` reads the accessibility tree (~10 tokens) and tells you every button/field on screen.  
`screenshot.sh` returns a base64 image (~1,600+ tokens).

**Always try `screen_map.sh` first.** Only take a screenshot when you need to visually verify UI layout or appearance.

---

## Scripts Reference

### 1. Health Check

```bash
bash scripts/health_check.sh
```

### 2. List Simulators

```bash
bash scripts/list_simulators.sh
# Returns JSON: [{name, udid, state, runtime}]
```

### 3. Boot a Simulator

```bash
# By name (partial match, case-insensitive)
bash scripts/boot.sh "iPhone 16 Pro"

# By UDID
bash scripts/boot.sh "A1B2C3D4-1234-1234-1234-A1B2C3D4E5F6"
```

### 4. Take Screenshot

```bash
# Default: half size (~1,600 tokens) — good for visual verification
bash scripts/screenshot.sh

# Quarter size (~800 tokens) — use when only checking layout
bash scripts/screenshot.sh --size quarter

# Full size — avoid unless you need pixel-perfect detail
bash scripts/screenshot.sh --size full

# Specific simulator
bash scripts/screenshot.sh --udid "iPhone 16 Pro" --size half
```

The output JSON contains `image_base64` (PNG). Render it as an image.

### 5. Read Screen Without Screenshot

```bash
# List all interactive elements (buttons, text fields) — very cheap
bash scripts/screen_map.sh

# Include coordinates for tapping
bash scripts/screen_map.sh --verbose
```

Output example:
```json
{
  "elements": [
    {"type": "button", "label": "Login"},
    {"type": "textfield", "label": "Email"},
    {"type": "button", "label": "Sign Up"}
  ]
}
```

Use element labels + `--verbose` coordinates to plan your next tap.

### 6. Tap / Swipe

```bash
# Tap at logical coordinates (iPhone 16 Pro viewport: 393×852 pts)
bash scripts/tap.sh 196 400

# Swipe (scroll down: start bottom → end top)
bash scripts/swipe.sh 196 600 196 200

# Swipe with custom duration
bash scripts/swipe.sh 196 600 196 200 --duration 0.5
```

### 7. Type Text

```bash
# Tap the field first, then type
bash scripts/tap.sh 196 300
bash scripts/type_text.sh "hello@example.com"
```

### 8. App Management

```bash
# Install a .app bundle
bash scripts/install_app.sh /path/to/MyApp.app

# Launch by bundle ID
bash scripts/launch_app.sh com.example.MyApp

# Both: install then launch
bash scripts/install_app.sh /path/to/MyApp.app && bash scripts/launch_app.sh com.example.MyApp
```

### 9. View Logs

```bash
# Last 50 lines from booted simulator
bash scripts/logs.sh

# Last 100 lines, errors only
bash scripts/logs.sh --lines 100 --severity error

# Filter to specific app
bash scripts/logs.sh --bundle com.example.MyApp
```

### 10. Status Bar (for clean screenshots)

```bash
# Set clean status bar before taking screenshots
bash scripts/set_status_bar.sh --preset clean   # 9:41, 100% battery, Wi-Fi

# Reset to default
bash scripts/set_status_bar.sh --clear
```

### 11. Flutter Operations

```bash
# Get dependencies
cd /path/to/flutter_project && flutter pub get

# Build for simulator (no code signing needed)
bash scripts/flutter_build.sh /path/to/flutter_project

# Build release
bash scripts/flutter_build.sh /path/to/flutter_project --mode release

# Run tests
bash scripts/flutter_test.sh /path/to/flutter_project

# Run specific test file
bash scripts/flutter_test.sh /path/to/flutter_project test/widget_test.dart

# Run app on simulator (interactive; Ctrl+C to stop)
bash scripts/flutter_run.sh /path/to/flutter_project
bash scripts/flutter_run.sh /path/to/flutter_project --udid "iPhone 16 Pro"
```

---

## Common Workflows

### Workflow A: Test a Flutter Feature End-to-End

```
1. bash scripts/health_check.sh
2. bash scripts/boot.sh "iPhone 16 Pro"
3. bash scripts/flutter_build.sh /path/to/project
4. bash scripts/install_app.sh /path/to/project/build/ios/iphonesimulator/Runner.app
5. bash scripts/launch_app.sh com.example.myapp
6. bash scripts/screen_map.sh          # read current screen
7. bash scripts/tap.sh <x> <y>         # tap a button
8. bash scripts/screenshot.sh --size half  # verify result
```

### Workflow B: Clean Screenshot for Docs / PR

```
1. bash scripts/set_status_bar.sh --preset clean
2. # Navigate to the desired screen
3. bash scripts/screenshot.sh --size half
4. bash scripts/set_status_bar.sh --clear
```

### Workflow C: Debug a Crash

```
1. bash scripts/launch_app.sh com.example.myapp
2. # Reproduce the crash
3. bash scripts/logs.sh --lines 200 --severity error --bundle com.example.myapp
```

### Workflow D: Run All Tests and Report

```
1. bash scripts/flutter_test.sh /path/to/project
# Output shows pass/fail for each test
```

---

## Coordinate System

- All coordinates are **logical points**, not pixels
- iPhone 16 Pro: **393 × 852 pts**
- iPhone 15 Pro Max: **430 × 932 pts**
- iPhone SE 3rd gen: **375 × 667 pts**

When you don't know coordinates:
1. Run `screen_map.sh --verbose` to get element frames
2. Compute center: `x = frame.x + frame.width/2`, `y = frame.y + frame.height/2`

---

## Troubleshooting

**"No booted simulator found"**
→ Run `bash scripts/boot.sh "iPhone 16 Pro"`

**"idb not found"**  
→ `brew install idb-companion && pip install fb-idb`
→ tap/swipe/screen_map require idb

**Flutter build fails**
→ Make sure Xcode is installed and agreed to license: `sudo xcodebuild -license accept`
→ Run `cd /project && flutter pub get` first

**xcrun simctl: No such file or directory**
→ Install Xcode from App Store, then `xcode-select --install`

---
> Source: [gabriel0952/ios-simulator-skill](https://github.com/gabriel0952/ios-simulator-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
