## deskflight

> Live flight radar on the **Waveshare ESP32-S3-Touch-AMOLED-1.43** (466×466 round

# DeskFlight — Claude working notes

Live flight radar on the **Waveshare ESP32-S3-Touch-AMOLED-1.43** (466×466 round
AMOLED, SH8601/CO5300 driver, FT3168 touch). Aircraft are pulled from the public
ADS-B feed and drawn as blips with a sweep line, range rings, heading arrows,
airline IATA codes, flight-level altitude, and a tap-to-inspect popup.

## Build environment

Arduino sketch (**not** PlatformIO — PlatformIO ships IDF 5.1.x which is
incompatible with the Waveshare display driver that needs IDF 5.5 APIs).

- Tool: **Arduino IDE 2.x** (or Maker Workshop)
- Platform: `esp32:esp32 3.3.8`
- Board: `Waveshare ESP32-S3-Touch-AMOLED-1.43`
- **Required board settings (non-negotiable, the device will brick or not boot):**
  - USB CDC On Boot: **Enabled**
  - Partition Scheme: **16M Flash**
  - PSRAM: **Enabled**
- Library: `lvgl 9.2.2` (install via Library Manager)
- All source files live at the **sketch root** (no `src/` directory) — Arduino IDE requirement.

## File layout

```
DeskFlight.ino           main sketch — setup(), loop(), networkTask
amoled.{cpp,h}           Amoled class (esp_lcd wrapper) — displayOn, invertColor, setRotation
low_level_amoled.{c,h}   Waveshare SH8601/CO5300 vendor driver (IDF 5.5)
FT3168.{cpp,h}           touch driver — hardened against I²C glitches
i2c.{c,h}                shared I²C bus (SDA=47, SCL=48) — FT3168 + QMI8658 IMU + PCF85063 RTC
board_config.h           Waveshare pin definitions
lv_conf.h                LVGL config — Montserrat 14/18/30/36 fonts enabled
config.h                 DeskFlight app constants (zoom levels, API hosts, NVS keys)
nvs_manager.{cpp,h}      AppSettings struct + NVS load/save
wifi_manager.{cpp,h}     STA connect + captive portal (sets 3 locations at once)
geocoding.{cpp,h}        location string → lat/lng via Nominatim
flight_api.{cpp,h}       adsb.lol free (plain HTTP) + ADS-B Exchange paid (HTTPS)
aircraft_manager.{cpp,h} aircraft state with dead-reckoning + LERP correction
radar_ui.{cpp,h}         LVGL canvas radar render + tap handlers + airline lookup
settings_carousel.{cpp,h} single-page-at-a-time settings overlay (8 tiles)
location_presets.{cpp,h} DEAD CODE — old overlay, safe to delete
```

## Current state (stable enough to ship as v0.9)

### Display

- **Hardware 90° clockwise rotation** via `Amoled::setRotation(90)` using
  `esp_lcd_panel_swap_xy` + `esp_lcd_panel_mirror`. USB-C sits at the bottom.
  Touch coords transformed in `lvgl_touchpad_read`: `(rx, ry) → (ry, W-1-rx)`.
- **Logical map orientation** (`map_orient` setting, N/E/S/W up) rotates the
  radar contents around CX/CY in software. Composes cleanly with the panel
  rotation above.

### Touch driver (FT3168.cpp)

Hardened over multiple iterations:
- Initialise buffers to zero (was the root cause of phantom touches — stack
  garbage from failed I²C reads was being interpreted as a touch)
- Check I²C return code on every read; any error → no touch
- Validate touch count in low nibble of reg 0x02; reject `> 5`
- Clamp coordinates at panel bounds (was rejecting frames, broke long-press)
- **120 ms hold-grace window** — single I²C glitch during a held finger keeps
  reporting the last valid coordinates so LVGL doesn't perceive a release.
  Without this, LV_EVENT_LONG_PRESSED_REPEAT counters reset every glitch and
  the RECONFIGURE hold never accumulated.

### Settings (8 tiles, single-page-at-a-time rendering)

Overlay creates persistent chrome (CLOSE, page title, `<`/`>` arrows, page
dots) once; `s_content` container gets `lv_obj_clean`'d and rebuilt on each
navigation. **Peak widget count ~25 (~15 KB internal SRAM), down from ~150
(~90 KB) in the original tileview design.** That rewrite fixed the OOM /
StoreProhibited panic that happened any time settings opened while the
network task was fetching.

Tiles in order:
0. AIRCRAFT LABELS — tap cycles NONE / CALLSIGN / FULL DATA
1. MAX AIRCRAFT — tap cycles 5/10/15/20/30/ALL; renders the N closest planes by partial-sort
2. ORIENTATION — tap cycles N/E/S/W up
3. DATA SOURCE — ADSB.LOL FREE / ADS-B PAID
4. PROX ALERT — big circle, tap ON/OFF
5. LOCATION — three slots, tap to switch (states: empty / pending / ready)
6. SETUP — shows current ssid + postcode; orange RECONFIGURE button (hold 2 sec)
7. SLEEP HOURS — wake/sleep ± controls; **call is commented out in loop() pending TZ support**

All event callbacks check `ignore_event()` (700 ms guard after `show()`) so
the tap that opened the overlay can't bleed through to a button at the same
pixel.

### Setup flow

1. First boot or after RECONFIGURE → captive portal AP `FlightRadar-Setup`
2. User types Wi-Fi creds + **3 location strings** (loc1 required, loc2/3 optional) + optional ADS-B Exchange RapidAPI key
3. Save → device reboots → `setup()` geocodes each non-empty preset via Nominatim (shows `"Locating N/3..."` splash per slot)
4. Lands on preset 0, radar starts
5. Settings → LOCATION → tap to switch any time

`networkTask` re-attempts geocoding every 30 s for any preset that still has
empty lat/lng (one preset per tick, polite to Nominatim).

### Flight API

- **Free path**: plain HTTP `http://api.adsb.lol/v2/lat/{lat}/lon/{lon}/dist/{nm}`
  via `WiFiClient` — no TLS, no mbedTLS allocations, no AES decryption pressure.
  Body buffered into PSRAM via `read_body_to_psram()` (6 s idle timeout), socket
  closed, **then** ArduinoJson parses from the PSRAM buffer. Completely
  decouples network read from JSON tokenizer heap activity.
- **Paid path**: ADS-B Exchange via RapidAPI, HTTPS-only. Same PSRAM body
  buffer pattern.
- Aircraft fields parsed: `hex`, `flight`, `lat`, `lon`, `alt_baro` (number or
  the string `"ground"` → `alt_m = -1.0f` sentinel), `gs`, `track`.

### Radar visuals

- 3 concentric rings at R/3, 2R/3, R; ring labels show km distance for current zoom
- Sweeping line with 15-step fading trail (~30 fps, 1.8°/frame, ~6 s/rotation)
- Compass labels (N/E/S/W) rotate per `map_orient`
- Aircraft blips with **heading arrows** (only when speed > 5 m/s)
- Labels: **ICAO → IATA airline lookup** (~70 carriers, e.g. `UAE202` → `EK202`)
- Altitude rendered as `FL%03d` above 10 000 ft, `%dft` below, `GND` for grounded
- **Cluster label de-overlap** — closest planes get label priority; later ones
  whose label would collide just show as dots
- Tap a blip → info popup (callsign, altitude, speed in km/h, heading)
- Proximity alert pulses for 4 seconds then fades out; only re-triggers if the
  area clears and a plane re-enters
- Top-left status: `X/Y AC` (visible/last-fetched), `12s ago` or `ERR H401` or `NO WIFI`
- Centered message when empty: `No WiFi` / `Locating: <postcode>` / `API error` / `No aircraft in area`

### Memory model

The device is designed to run indefinitely without manual resets:

- **Aircraft array is fixed-size** (40 slots × ~100 bytes). Slots reuse by
  `icao24`. Stale slots inactive after `AIRCRAFT_TTL_MS=90s` via `pruneStale()`.
  Trails are fixed-length per plane (`TRAIL_LEN=5`).
- **JSON parsing is bounded per fetch.** ArduinoJson 7 pool in PSRAM,
  `ps_malloc` + `free` each fetch.
- **HTTP state is stack-scoped.** `HTTPClient` + `WiFiClient[Secure]` live in
  the fetch function.
- **LVGL canvas + draw buffers** in PSRAM, fixed.
- **Settings widgets** allocated on show, destroyed on hide. Peak ~15 KB
  internal SRAM during settings open.

`networkTask` logs `[HEAP] int free=N (largest=M, delta=±D) | psram free=P |
uptime=S` every 60 s. **Critical gotcha:** `ESP.getFreeHeap()` returns
**combined** heap on ESP32-S3 with PSRAM, which hides internal-SRAM
exhaustion. Always use `heap_caps_get_*(MALLOC_CAP_INTERNAL)` for accurate
diagnostics.

### Useful Serial tags

- `[NET]` — network task fetch loop
- `[API]` — flight API requests
- `[GEO]` — geocoding requests
- `[TAP]` — touch events on the radar canvas
- `[SET]` — settings overlay show/hide/navigate
- `[HEAP]` — periodic heap report
- `[PORTAL]` — captive portal save events

### Recovery

If anything gets stuck — bad NVS, wrong WiFi, broken location — open settings,
swipe to **SETUP**, **hold** the orange RECONFIGURE button for ~2 seconds.
Wipes WiFi credentials + location and reboots → captive portal reopens. All
other settings (presets' labels are wiped, but theme/API mode/etc.) survive.

If even settings is unreachable: in Arduino IDE, **Tools → Erase All Flash
Before Sketch Upload → Enabled**, upload, then set it back to Disabled. Wipes
NVS entirely.

## Open work for v1.0 — stability sweep (next session)

### High impact — must-do before shipping

**1. Active WiFi reconnect logic**
Right now if WiFi drops, the device shows `NO WIFI` and waits for ESP32's
built-in auto-reconnect, which isn't reliable. Should explicitly check
`WiFi.status()` in `networkTask` and after N consecutive failed fetches call
`WiFi.disconnect()` + `WiFi.begin(ssid, pass)` to force a fresh association.
Periodic kick every few minutes if still disconnected. No-op cost when WiFi
is healthy.

**2. Heap watchdog auto-action**
We log `[HEAP]` but never act on it. If `largest contiguous block (MALLOC_CAP_INTERNAL)`
drops below ~25 KB the next TLS handshake / large alloc will fail. Should
schedule a graceful reboot at that point — save NVS, log `[HEAP] CRITICAL:
auto-reboot`, then `ESP.restart()`. Insurance against the heap-fragmentation
death spiral. Hopefully never fires given the plain-HTTP architecture.

**3. Geocode failure UX**
If a preset's city string is unresolvable (typo, country missing, etc.) the
LOCATION page shows `(locating...)` in yellow forever and the network task
keeps retrying every 30 s indefinitely. Should track retry count per preset
in memory, mark as `"failed"` after ~10 attempts, show it differently in the
LOCATION page (red, untappable, `not found`), and stop the retry loop until
the preset is re-entered via RECONFIGURE.

### Medium impact

**4. Sleep schedule re-enable with timezone**
Code exists in `DeskFlight.ino` but the `apply_sleep_schedule()` call is
commented out. To re-enable:
- Add a `tz_offset` int field to AppSettings (NVS key `tz_off`)
- Add to captive portal HTML as a `<select>` of common UTC offsets (-12 to +14)
- Pass `tz_offset * 3600` as the first arg to `configTime()`
- Uncomment the call in `loop()`
- Test wake/sleep boundary at a known wall-clock time
~30 min of work.

**5. Longitude-cosine bug in proximity distance**
In `radar_tick()` close-call calc:
```cpp
float dist_km = sqrtf(dlat*dlat + dlng*dlng) * 111.32f;
```
treats longitude as 111 km/° regardless of latitude. Actual east-west
distance shrinks with `cos(lat)`. Result: proximity alert under-triggers on
the east-west axis at high latitudes. One-line fix:
```cpp
float kmlon = 111.32f * cosf(s_settings->lat * (float)M_PI / 180.0f);
float dist_km = sqrtf(dlat*dlat*111.32f*111.32f + dlng*dlng*kmlon*kmlon);
```

### Low impact — cleanup & polish

**6. Delete `location_presets.{cpp,h}`**
Dead code, no longer included anywhere. The class was replaced by the
LOCATION tile in the settings carousel. Safe to delete + remove from any
mention in this file.

**7. Boot crash counter in NVS**
Increment a counter on every `setup()`, log it. `[BOOT] count: 47` would be
a smoking gun if the user reports flakiness — if it grows fast, the device
has been crash-rebooting silently. New NVS key `boot_cnt`, one `getUInt` +
`putUInt` per boot.

**8. Aircraft mutex hold-time audit (low risk)**
Render thread holds `aircraftMgr.mutex` while iterating 40 entries + drawing
labels. Usually <1 ms, but worst case could stall the network task's
`updateFromAPI`. Could split into a "read-only snapshot copy" pattern, but
not currently a problem in practice. Audit only if profiling shows it.

### Suggested order of attack for next session

1. **1, 2, 3** as one "stability sweep" commit + PR
2. **4** as a feature commit
3. **5** as a one-line correctness fix
4. **6, 7** as a cleanup commit

Then tag `v1.0` and ship.

## Conventions

- Keep changes inline; no new abstractions until ≥3 near-duplicate uses exist.
- No emojis in code or commit messages.
- Commit messages: HEREDOC body + `Co-Authored-By: Claude` trailer.
- New features expose Serial logging (`[TAG] …`) so the user can diagnose
  without UI access.
- On-screen status over silent failures — when something doesn't work the
  radar should *say* why.

## Webserver String lifetime gotcha (learned the hard way)

Arduino `WebServer::arg("name")` returns a `String` **by value**. Calling
`.c_str()` on the temporary is safe *only if* the result is consumed within
the same full expression. This is dangling:

```cpp
const char *l1 = server.arg("loc1").c_str();   // String destroyed at ;
strlcpy(buf, l1, n);                            // UB — l1 dangles
```

This is fine (because the temporary lives through the full expression):

```cpp
strlcpy(buf, server.arg("loc1").c_str(), n);
```

This is also fine (the named String lives for the scope):

```cpp
String l1 = server.arg("loc1");
strlcpy(buf, l1.c_str(), n);
```

---
> Source: [elm1nst3r/DeskFlight](https://github.com/elm1nst3r/DeskFlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
