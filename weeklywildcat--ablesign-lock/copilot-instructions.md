## ablesign-lock

> This repository contains the Android TV launcher and recovery client for AbleSign.

# Wildcat Kiosk Android context

This repository contains the Android TV launcher and recovery client for AbleSign.

- Validate Android changes with `./gradlew assembleDebug`.
- Preserve the sleep policy: do not add `FLAG_KEEP_SCREEN_ON`, wake locks, foreground services,
  or exact alarms.
- The kiosk uses an accessibility service to recover AbleSign and a timed admin menu for staff.
- Unpaired or revoked devices show Health Pairing automatically. Paired devices can reopen it with
  a long Select/OK press while the Wildcat logo is visible; retain this Fire TV/Google TV gesture.
- The Android client reports to the standalone Signage Health service at
  `https://github.com/weeklywildcat/signage-health`; server, Docker, and Cloudflare changes belong
  there, not in this repository.
- Do not commit APKs, Gradle caches, local device configuration, or any health-service tokens.

---
> Source: [weeklywildcat/ablesign-lock](https://github.com/weeklywildcat/ablesign-lock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
