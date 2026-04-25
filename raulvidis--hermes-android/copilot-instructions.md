## hermes-android

> This extension adds Android device control to hermes-agent via the `android` toolset.

# hermes-android

## Overview
This extension adds Android device control to hermes-agent via the `android` toolset.
It communicates with a bridge app running on an Android device over HTTP.

## Setup
1. Install the bridge APK on the Android device
2. Grant the bridge app Accessibility Service permission in Settings > Accessibility
3. Grant SYSTEM_ALERT_WINDOW permission
4. Set ANDROID_BRIDGE_URL in ~/.hermes/.env (only needed for direct USB/LAN connection):
   - Same WiFi: `ANDROID_BRIDGE_URL=http://192.168.x.x:8765`
   - USB (recommended): run `adb forward tcp:8765 tcp:8765` then use `http://localhost:8765`
   - Remote relay (default): no config needed — `android_setup` starts a relay on port 8766
5. Install the Python package: `pip install -e ./hermes-android`
6. Add to hermes-agent model_tools.py _modules: `"tools.android_tool"`
7. Add "android" toolset to toolsets.py

## Tool usage patterns

### Read before act
ALWAYS call android_read_screen before tapping. Never guess coordinates.

### Prefer text over coordinates
Use android_tap_text("Continue") over android_tap(x=540, y=1200).

### Wait after navigation
After opening an app or tapping a button that triggers loading,
always call android_wait with expected text before next action.

### Confirmation pattern for destructive actions
Before confirming a purchase, ride, or send action — always report
to the user what you're about to do and wait for approval.
Example: "I'm about to confirm an Uber ride to [destination] for [price].
Reply 'yes' to confirm."

## Common package names
- com.ubercab — Uber
- com.bolt.client — Bolt
- com.whatsapp — WhatsApp
- com.spotify.music — Spotify
- com.google.android.apps.maps — Google Maps
- com.android.chrome — Chrome
- com.google.android.gm — Gmail
- com.instagram.android — Instagram
- com.twitter.android — X/Twitter

---
> Source: [raulvidis/hermes-android](https://github.com/raulvidis/hermes-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
