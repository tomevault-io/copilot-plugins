## eggubator

> Single-file Arduino firmware for a DHT22-based egg incubator with web UI, flash data logging, and OTA updates. All HTML/CSS/JS lives inside `web_ui.h` as PROGMEM string literals — edit raw HTML embedded in C.

# AGENTS.md — Egg Incubator (ESP8266)

Single-file Arduino firmware for a DHT22-based egg incubator with web UI, flash data logging, and OTA updates. All HTML/CSS/JS lives inside `web_ui.h` as PROGMEM string literals — edit raw HTML embedded in C.

## Build & Deploy

```bash
# Compile
arduino-cli compile -b esp8266:esp8266:nodemcu -j "$(nproc)" --build-path build/.cache --output-dir build eggubator.ino
cp build/eggubator.ino.bin firmware.bin

# Deploy OTA (user insists on this script only)
./deploy.sh                          # compile + find IP via ping + curl to /update

# Flash USB (Termux/OTG)
./flash.sh                           # auto-detect /dev/ttyUSB*, uses esptool.py

# Full release (bump → compile → commit → tag → GitHub Release → OTA)
./rel.sh [VERSION]                   # auto-increment patch if no VERSION arg
./rel-nd.sh [VERSION]                # same without OTA deploy
```

- `rel.sh` bumps `updates.h` + `version.txt`, compiles, creates GitHub Release with `firmware.bin`, then deploys OTA.
- `firmware.bin` is `.gitignore`d but release scripts `git add` it explicitly.
- `version.txt` must match `FIRMWARE_VERSION` in `updates.h`.

## Hardware Conventions

| Component | Pin | Active |
|-----------|-----|--------|
| Heater    | D1  | LOW = ON |
| Atomizer  | D2  | LOW = ON |
| Fan       | D3  | LOW = ON |
| Servo     | D5  | PWM (544-2450µs) |
| DHT22     | D4  | — |

**Relays are active LOW.** `digitalWrite(pin, HIGH)` = OFF, `LOW` = ON. Getting this wrong can overheat or damage hardware.

Network: mDNS `eggubator.local`, static IP ends in `.72`. Falls back to AP mode (`EGGubator` SSID).

## WiFi — Async State Machine (not blocking)

`wifi_manager.h` uses 5 states — no blocking `delay()` in init:

```
WF_TRY_SAVED → (10s timeout) → WF_TRY_DEFAULT → (10s timeout) → WF_AP
    ↕                               ↕                                  ↕
WF_CONNECTED ← (auto-reconnect) → WF_RECONNECTING (15s grace)   scan every 15s for saved/default SSID
```

- Boot order: `EEPROM.begin(512)` → `loadWifiCredentials()` → `initWiFi()` (non-blocking). Must stay in this order.
- `initWiFi()` returns immediately; `handleWiFi()` drives state from `loop()`.
- Priority: saved EEPROM creds → compile-time defaults (`config.h`) → AP fallback.
- AP mode runs a DNS captive portal and scans every 15s for known networks.
- `MDNS.begin()` deferred to first WiFi connection in loop (not setup).
- Saving WiFi creds via web (`/settings/api?wifiSsid=X&wifiPassword=Y`) writes EEPROM at addr 200 — does NOT reconnect. User must reboot.

## EEPROM Layout

| Region | Address | Magic | Size |
|--------|---------|-------|------|
| SAT drift | 15-23 | — | 9 bytes |
| DeviceSettings | 40 | `0xAB` | 112-byte struct |
| WiFi credentials | 200 | `0xAC` | `WifiSettings` (98 bytes) |

- `loadWifiCredentials()` reads addr 200 — if magic invalid or SSID empty, keeps compile-time defaults.
- `saveWifiCredentials()` writes addr 200.
- Flash logging uses **separate** circular buffer at `0x200000` (256 sectors × 4096 bytes) — EEPROM untouched by logging.

## Verification

1. **Compile** — must succeed with zero errors.
2. **Browser** — `http://eggubator.local/`, dashboard loads, `/status` returns JSON.
3. **Manual Playwright tests** at `test/playwright/test_*.js` — `node test_xxx.js` (device must be reachable).
4. **Mock mode** (no hardware): `/settings/api?enable=1&temp=37.5&hum=55` or `/settings/api?autosim=1`.

## Embedded Assets (no internet needed)

5 gzipped libraries compiled into firmware via `embedded_assets.h`:

| Asset | Path |
|-------|------|
| Chart.js 4.4.7 | `/lib/chartjs/chart.umd.min.js` |
| Dexie.js 3.2.2 | `/lib/dexie/dexie.min.js` |
| Hammer.js | `/lib/hammerjs/hammer.min.js` |
| chartjs-plugin-zoom | `/lib/chartjs-plugin-zoom/chartjs-plugin-zoom.min.js` |
| Bootstrap 5 CSS | `/lib/bootstrap/css/bootstrap.min.css` |

- Served with `Content-Encoding: gzip` and `Cache-Control: public, max-age=31536000, immutable`.
- Registered as routes in `setup()` via `EMBEDDED_ASSETS` table.
- Web UI (`web_ui.h`) loads these paths — they resolve locally, not from CDN.

## SVG Icons & Animation Quirks

- 4 stat cards (heater, atomizer, fan, servo) use **Font Awesome solid SVG paths** with `fill="currentColor"`.
- **SVG class assignment must use `setAttribute('class', ...)`** not `.className =` — SVG's `SVGAnimatedString` silently ignores string assignment.
- Fan rotation uses CSS animation `.svg-spin` with `transform-box: fill-box` for reliable centering.

## File Map

| File | Purpose |
|------|---------|
| `eggubator.ino` | Setup/loop, web handlers, auto-control, EEPROM save/load |
| `config.h` | Pin defs, compile-time WiFi defaults, hysteresis |
| `dht_sensor.h` | DHT22 read + physics simulation (mock/auto-sim) |
| `wifi_manager.h` | Async WiFi state machine + DNS captive portal |
| `logging.h` / `.cpp` | Flash circular buffer (256 sectors at `0x200000`) |
| `sat_manager.h` / `.cpp` | Boot session tracking, absolute time recovery across reboots |
| `updates.h` | OTA check + download from GitHub releases |
| `web_ui.h` | Single file: all HTML/CSS/JS as PROGMEM strings |
| `embedded_assets.h` | 5 gzipped JS/CSS libs compiled in (no CDN) |
| `sector_viewer.h` | Flash hex editor tool |

## Key Architecture Notes

- **Timing globals** (`LOG_INTERVAL`, `EGG_TURN_INTERVAL`, `PULSE_ON/OFF_TIME`, `TARGET_TEMP/HUMIDITY`) are web-modifiable, not compile-time constants.
- **SAT**: browser syncs timeline across power cycles via `/timestamps` (GET/PUT). `batchStartUnix` + `getElapsedSeconds()` = incubation day.
- **Servo**: 32 steps × 6°, center at step 15 (90°). Stage lockdown (day 18+) disables turning and moves servo to center.
- **DHT fallback**: if `isnan()`, returns last valid reading; temp/hum validation also requires > 0.
- **Servo pin (D5=GPIO14) held LOW during boot** to suppress SPI noise — must happen before any other pin init on GPIO14.
- **No external Arduino libraries beyond ESP8266 core** + `Servo.h`, `DHT.h`. DHT has local bit-banged fallback.
- **No CI, no linter, no formatter config.**

## Web Endpoints

| Path | Method | Purpose |
|------|--------|---------|
| `/status` | GET | JSON: temp, humidity, device states, version, boot info |
| `/data` | GET | JSON sensor log (pagination via `boot`, `time`, `count`) |
| `/settings/api` | GET/POST | Mock/autosim, timing, stage, servo angles, WiFi creds |
| `/control?device=X&mode=Y` | GET | `off` = kill override, anything else = auto |
| `/settings/clear` | GET | Erase flash logs, reset boot ID, reboot |
| `/ota/check` | GET | Compare version vs GitHub release |
| `/ota/apply` | POST | Download + flash firmware.bin from GitHub |
| `/timestamps` | GET/PUT | SAT boot table sync |
| `/reboot` | GET | `ESP.restart()` |
| `/update` | POST | ESP8266HTTPUpdateServer (used by `deploy.sh`) |

## Connectivity (when device unreachable)

1. `http://eggubator.local`
2. `ping -c 1 eggubator.local` or `arp-scan -l`
3. `http://192.168.X.72` (X from DHCP subnet)
4. If all fail, report unreachable — no automated retries.

## Gotchas

- `config.h` has actual WiFi credentials — don't commit changes to it.
- `firmware.bin` is gitignored but release scripts stage it explicitly.
- `wifiSsid`/`wifiPassword` globals in `wifi_manager.h` are runtime-writable — EEPROM loads over defaults on boot if valid.
- Clearing WiFi creds (empty SSID via web) resets to compile-time defaults in `config.h`.
- SVG `className` assignment fails silently — always use `setAttribute('class', ...)`.
- `EEPROM.begin(512)` must happen before `loadWifiCredentials()` and `initWiFi()`.
- mDNS blocked on mobile hotspots — device unreachable via `.local` when connected through phone hotspot.
- Firmware IRAM ~94% full (~385KB IROM headroom) — tight on `.text` sections.
- No automated test runner. No CI. No formatter. No linter.

---
> Source: [Vinayrnani/eggubator](https://github.com/Vinayrnani/eggubator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
