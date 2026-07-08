## 30-firmware

> ESP32-S3 amenities post-controller firmware. Apply when touching firmware/** (PlatformIO C++). Different toolchain and runtime from the web app — do not carry web/TS assumptions in here.


# Amenities post-controller firmware (ESP32-S3, PlatformIO)

This is the **client side** of the amenities device contract. It lives in `firmware/amenities-post-controller/` in this repo, but it is **C++ on PlatformIO**, not the React/Deno web app — do not assume web tooling, npm, or TypeScript here. Treat the web app and the firmware as two implementations of one shared contract.

## The shared contract is the source of truth
- `firmware/amenities-post-controller/docs/server-contract.md` is the **single source of truth** for the device↔server interface. The edge functions (`amenities-activate`, `amenities-member-cache`, `amenities-events-sync`, `amenities-heartbeat`, `amenities-help`) and this firmware must both match it.
- **Any change to the contract is an atomic PR that touches all three: the contract doc, the edge function(s), and the firmware.** Never change one side silently. If a slice changes a request/response shape, update `server-contract.md` first and call it out.
- The device speaks **raw HTTPS `POST https://<ref>.supabase.co/functions/v1/<fn>`** with its own bearer key — it does **not** use `supabase.functions.invoke` (that's the JS SDK).

## Device auth (client side of B.7)
- The device holds a **per-device bearer key**, flashed at provisioning and stored in `secrets.h` (gitignored; `secrets.example.h` is the template). The server stores only `device_key_hash` and checks `amenities_device.revoked` on every call — so a `403` can mean revoked, not just a bad key.
- **Auth-lockout (review C3):** back off after **5 consecutive** `401/403` responses; reset the counter on any `2xx`.
- Hold only the minimal cache and never log the bearer key or any phone digits.

## Offline-first behavior (the whole point of the post)
- The post **must work offline** and **must not call FlexWash**. Grants come from the **on-controller flash cache** (LittleFS/KV), an active-member subset.
- **Member cache (`amenities-member-cache`):** the response is **NDJSON of per-device salted 64-bit hashes** — `SHA-256(cache_salt ‖ national10)` truncated to 8 bytes, hex — **never phone digits**. The stream ends with a trailer line `{"end":true,"count":N}` and a `Content-Length` header (no chunked encoding). **Verify the stream is complete and count-matched before committing the delta** — a dropped connection must never half-apply. An offline grant from the cache is recorded as `offline_granted`.
- **Offline journal (`amenities-events-sync`):** queue events while offline; on reconnect, batch-upload. Each event carries a device-generated **`event_id` = boot-counter + per-boot sequence**; the server is idempotent on **`(device_id, event_id)`**. Offline activation entries carry **`phone_hash`** (the same salted cache hash), **never plaintext `phone_e164`**. `occurred_at` may be empty before NTP sync (server stamps receipt). **Treat any `2xx` as success** and clear the queue.
- **Heartbeat (`amenities-heartbeat`, ~5 min):** fire-and-forget, `hb_schema: 2`, includes `last_sync_age_s` and the health counters (`rssi`, `min_rssi`, `channel`, `bssid`, `cache_count`, `queued_count`, `uptime_s`, `wifi_disconnects`, `wan_failures`, `wan_auth_fails`, `last_wan_ok_s`). It is **health-only and never gates a grant** — a missing/forged heartbeat can only mislabel health.

## Keypad / HELP / display
- **HELP button:** dedicated **illuminated button on GPIO8** (v0.14) — this replaced the old double-`*` gesture. Keypad: **`*` = per-digit backspace, `#` = submit.** A help press creates a help card only (touches no grant state).
- **Pin map must stay in sync** across `src/config.h` / `src/hal.h`, the wiring SVG, and the build guides: relays 6/7/15–18, Wiegand 4/5, TFT SPI 9–14, HELP 8. No strapping/USB/flash-pin conflicts. If you change a pin, change all of them in the same PR.
- **Display:** Newhaven NHD-2.8 (rgb565 assets, `copy.h` strings ↔ the LCD mockup panels). Keep the 14 `copy.h` strings aligned with `Post LCD Screen - Messages & Mockups.html`.

## ✋ HOLD 2 — ADA / member accessibility (unresolved)
Do **not** bake in keypad/HELP **mounting height or reach** (ADA 308/309), one-hand-operation assumptions, or the **low-vision masked-entry UX** on the 2.8″ display without Tom's recorded decision. Leave these as flagged TODO constants. This must be decided **before Margate fabrication.**

## Build / CI
- PlatformIO project; keep its build as a **separate CI job** from the web/Deno pipeline. Don't add firmware deps to the web package manifests or vice-versa.
- `secrets.h` stays gitignored; commit only `secrets.example.h`.

---
> Source: [tderi33/spec-compliant-kit](https://github.com/tderi33/spec-compliant-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
