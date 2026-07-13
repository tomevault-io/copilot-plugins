## geek-instinct

> Garmin Instinct 2 Solar watch face built with Monkey C and Connect IQ SDK 9.1.0.

# AGENTS.md

Garmin Instinct 2 Solar watch face built with Monkey C and Connect IQ SDK 9.1.0.

## Target Device

| Parameter  | Value              |
| ---------- | ------------------ |
| Product ID | `instinct2` |
| Resolution | 176 × 176 px       |
| Display    | MIP 2-color (B&W)  |
| Solar      | Yes                |
| API Level  | 3.4.0              |

## Setup

Connect IQ SDK 9.1.0 is installed at:

```
~/Library/Application Support/Garmin/ConnectIQ/Sdks/connectiq-sdk-mac-9.1.0-2026-03-09-6a872a80b/bin/
```

The binaries (`monkeyc`, `connectiq`, `monkeydo`) are not in PATH. Use full paths or add the `bin/` directory to `~/.zshrc`.

```bash
SDK=~/Library/Application\ Support/Garmin/ConnectIQ/Sdks/connectiq-sdk-mac-9.1.0-2026-03-09-6a872a80b/bin

# Compile
$SDK/monkeyc -f monkey.jungle -o bin/geek-instinct.prg -d instinct2 -y developer_key.der

# Run on simulator
$SDK/connectiq
$SDK/monkeydo bin/geek-instinct.prg instinct2
```

## Warnings

**MIP 2-color display:** Only `Graphics.COLOR_WHITE` and `Graphics.COLOR_BLACK` render correctly. Other color constants silently map to one of these two — do not use them.

**Solar intensity:** `System.getSystemStats().solarIntensity` returns `null` on non-solar devices. Always null-check before arithmetic.

**Layout:** Never hardcode pixel coordinates. Read dimensions from `dc.getWidth()` / `dc.getHeight()` in `onLayout()`.

**Permissions:** Declare every API namespace used (`SensorHistory`, `UserProfile`, etc.) in `manifest.xml` under `<iq:permissions>`. Undeclared APIs throw runtime errors.

**Performance:** Use `onPartialUpdate()` with `dc.setClip()` for elements that change every second. Allocate objects as class-level `var` fields, not inside `onUpdate()`.

**Subscreen:** Instinct 2 has a physical circular cutout at top-right. Get its rect via `WatchUi.getSubscreen()` and draw HR inside it. Avoid placing other UI elements behind the subscreen area.

---
> Source: [Skesov/geek-instinct](https://github.com/Skesov/geek-instinct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
