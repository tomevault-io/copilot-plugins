## mqtt-broker-esp

> This file is read first by AI coding agents (Claude, Cursor, Aider, etc.)

# AGENTS.md — working with this repo as an AI coding agent

This file is read first by AI coding agents (Claude, Cursor, Aider, etc.)
before they make changes. It documents the conventions a human contributor
would learn the hard way over a few PR cycles. Follow it.

If something here disagrees with `README.md` or `docs/`, the human-facing
docs win — update this file to match, don't drift.

---

## 1. What this project is

A standalone **MQTT 3.1.1 broker** that runs entirely on an ESP32-S3,
written in C against ESP-IDF v5.5. No external MQTT library — the broker
is a single C codebase using lwIP BSD sockets. Ships a Tasmota-style web
portal, an SNTP server, Tasmota-style scheduled-publish timers, and an
optional Ethernet+NAPT gateway mode.

**Design goal:** 10-year deployment lifetime on a $10 chip with zero
maintenance. Every change should preserve that — see §7 ("Constraints").

---

## 2. Build & deploy loop

The canonical iterate-on-the-device workflow is:

```bash
# 1. Enter ESP-IDF environment (once per shell)
source $IDF_PATH/export.sh
# or, if IDF_PATH isn't set:
source /home/spencerkittleson/lvgl_micropython/lib/esp-idf/export.sh

# 2. Build
idf.py build
# Output binary: build/mqtt_broker.bin
# Track binary size against the previous tag — see §7.

# 3. Deploy via OTA to the live test device
PORTAL_AUTH=support:dockyard make ota
# Wraps: fetch /api/csrf with auth → POST build/mqtt_broker.bin to
# /ota-upload?csrf=<token>. Device auto-reboots on success.

# 4. Verify it came back on the new firmware
sleep 12 && curl -s -u support:dockyard http://192.168.22.100/api/status \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['firmware'])"
```

**Live test device:** `http://192.168.22.100/` (Waveshare ESP32-S3-ETH).
Portal Basic Auth: `support:dockyard`. Treat as the source of truth for
"does this actually work" — `idf.py build` succeeding is necessary but
not sufficient.

**Never flash via USB** in normal iteration. OTA is the canonical path
because it's what users will use, and it catches regressions (partition
table, app-format, rollback) that USB flash skips.

---

## 3. Code map

```
main/
  main.c            Entry. WiFi → portal → ntp → timers → broker, in that order.
  mqtt_broker.{c,h} The broker. Owns clients[], subs[], retained store, $SYS publishes,
                    and the thread-safe publish queue (broker_publish_local /
                    tester pub queue). Single broker_task — almost everything
                    runs in select() loop.
  mqtt_parser.{c,h} Wire-format encode/decode for the MQTT 3.1.1 packets.
  portal.{c,h}      The HTTP server. ~3700 lines. Route dispatch by string
                    match on req.path. CSRF + Basic Auth on all gated routes.
                    Page-render uses snprintf into a 16 KB heap buffer.
  portal_ws.{c,h}   /tester WebSocket (live MQTT subscribe + publish via UI).
  ntp.{c,h}         SNTP client + SNTPv4 server. Owns the POSIX TZ env via
                    setenv("TZ", ns.tz, 1) + tzset() at boot.
  timers.{c,h}      Tasmota-style scheduled publishes (16 slots, 1 Hz task).
                    Stores a compact JSON blob in NVS "mqtt_cfg"/"timers".
                    Resolves local time via newlib's localtime_r so DST is free.
  tz_presets.{c,h}  Curated ~40-entry table of POSIX TZ strings for the
                    /settings dropdown. Hand-maintained from IANA tzdata.
  wifi_connect.{c,h} WiFi station + AP. Owns NVS "mqtt_cfg" credentials.
  eth_connect.{c,h} Optional W5500 SPI Ethernet + NAPT (CONFIG_MQTT_BROKER_ETHERNET).
  csrf.{c,h}        Per-boot random token. csrf_verify(req.csrf_header, req.body)
                    is required for every state-changing request.
  version.h         FW_VERSION, FW_NAME. Bumped per release.

tools/              Python scripts (Playwright/pytest helpers).
  capture_*.py      Headless screenshot generators → docs/screenshots/.
                    Always parameterized via PORTAL_URL + PORTAL_AUTH env.

tests/              (none yet — test_broker.py / test_ntp.py live at repo root)
test_broker.py      pytest. Drives mosquitto_pub/sub against the live device.
test_ntp.py         pytest. Asserts SNTP client + server behaviour.

docs/
  api.md            Path/method/auth table + JSON schemas + curl examples.
  architecture.md   Task layout, memory map, partitions, scaling notes.
  screenshots/      Portal UX screenshots, grouped by feature subdir.
  *-audit-*.md      Per-version UX audit reports + fix sequencing.
plan-*.md           Design docs for upcoming features. Written before code.
changelog/          One file per release: CHANGELOG-v<X.Y.Z>.md.
CHANGELOG.md        Top-level summary that points at changelog/*.
```

---

## 4. NVS namespaces (avoid clobbering)

Read-write zones with their owners. Do not write keys outside an
established namespace without updating this table.

| Namespace  | Owner          | Notable keys                                                                                                                                                                |
| ---------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `mqtt_cfg` | portal.c       | `hostname`, `mqtt_port`, `auth_user`, `auth_pass`, `buf_size`, `retain_en`, `retain_ttl`, `ap_ssid`, `ap_pass`, `ap_ip`, `napt_en`, `timers` (JSON blob, owned by timers.c) |
| `ntp`      | ntp.c          | `enabled`, `srv_enabled`, `upstream_0..2`, `poll_s`, `tz`, `accept_set`                                                                                                     |
| `wifi`     | wifi_connect.c | `ssid`, `password`                                                                                                                                                          |

**The POSIX TZ string lives in `ntp/tz`**, not `mqtt_cfg`. All consumers
(portal, timers, NTP server, `$SYS/broker/time`) read it via the same
`setenv("TZ",...)` + `tzset()` path set up in `ntp_init()`.

---

## 5. Plan → implement → release rhythm

This repo runs on a strict "plan first" discipline. Look at
`plan-mqtt-ux-v2.md`, `plan-scheduled-publishes.md`, `plan-ntp-server.md`
for the template.

### 5.1 Before writing code

1. Write `plan-<feature>.md` at the repo root. Include:
   - Goals & non-goals.
   - Data model (NVS keys, wire format, struct shape).
   - HTTP/API surface table.
   - Phasing with effort + flash-budget estimate per phase + risk level.
   - Test strategy (unit + Playwright + pytest).
   - Open questions for the human to weigh in on.
2. Ask the user clarifying questions (use the questionnaire tool if you
   have one) **before** opening any code editor. Common axes: storage
   backend, max counts, UI surface placement, defaults.

### 5.2 While implementing

- One feature = one commit. Don't bundle unrelated dirty files even if
  they're in `git status`. The repo regularly has pre-existing local
  edits (`main/ntp.c`, `main/ntp.h`, `plan-*.md`) that are NOT part of
  the feature you're working on — `git add` only the files you touched.
- Reuse existing infrastructure aggressively. Example: the timer
  scheduler reuses the **tester publish queue** in `mqtt_broker.c` rather
  than adding a second producer path. Audit first, build second.
- Style is C99, 4-space indent, K&R brace style, `static` everything that
  doesn't need external linkage. Match the surrounding file.
- HTML/CSS in `portal.c` is **server-rendered via `snprintf`** into the
  16 KB `body` buffer. No template engine, no client framework. JS is
  inline `<script>` and must be optional — every form has to work
  without JS (graceful degradation matches the Tasmota soul).

### 5.3 After implementing

1. `idf.py build` must complete with no new warnings beyond the
   pre-existing `-Woverride-init` noise in `portal.c:262-265` (the
   base64 decode table).
2. **OTA-deploy to the live device** and run a smoke test that exercises
   the new surface end-to-end (HTTP API, MQTT publish, UI render).
   `make ota` then `curl /api/...` is the minimum bar.
3. **Re-capture screenshots** if the UI changed. Use the matching
   `tools/capture_*.py` script; commit the PNGs under
   `docs/screenshots/<feature>/`.
4. Write `changelog/CHANGELOG-v<X.Y.Z>.md`. Format:
   - Headline: what this release is, in one sentence.
   - Bullets grouped by P0/P1/P2 (correctness / significant UX / polish)
     or by subsystem.
   - "Out of scope" / "Backlog" section for items intentionally deferred.
   - Flash impact (binary delta in bytes, OTA-slot free %).
   - Verification trail (what was tested live).
5. Update top-level `CHANGELOG.md` with a paragraph summary that links
   to the detailed changelog file.
6. Bump `main/version.h`:
   - **Patch (`0.8.0 → 0.8.1`)**: bug fix, UX polish, additive API, no
     storage schema change.
   - **Minor (`0.7.x → 0.8.0`)**: new user-visible feature.
   - **Major (`0.x.y → 1.0.0`)**: storage schema change requiring
     migration, or a known-breaking compatibility change.
   - Always update the multi-line block comment above the `#define`s
     to describe what's new in this version. Future-you reading this in
     a year will be grateful.

### 5.4 Commit + tag

```bash
git add <only the feature files>
git commit -m "<type>(<scope>): <one-line summary>

Multi-paragraph body explaining what + why. Bullet sub-points for
significant items. Reference the plan doc, audit doc, or upstream
spec where relevant.

Flash impact and OTA verification notes go here."

git tag -a v<X.Y.Z> -m "v<X.Y.Z> — <one-line summary>

Slightly longer release-notes paragraph. Mirror the headline of the
matching changelog/CHANGELOG-v<X.Y.Z>.md file."
```

Commit `type` matches conventional commits: `feat`, `fix`, `docs`,
`refactor`, `test`, `chore`. Scope is the subsystem (`timers`,
`portal`, `broker`, `ntp`, etc.). Don't push until the user explicitly
asks — the conversational style here is "implement, OTA, verify, then
ask permission to push".

### 5.5 Push

```bash
git push --follow-tags origin main
```

The remote (`origin`) is `https://github.com/skittleson/mqtt_broker_esp.git`,
**a private repo under the `skittleson` GitHub account**. If `gh auth status`
shows `spencerkittleson` as the active user, switch first:

```bash
gh auth switch --hostname github.com --user skittleson
gh auth setup-git    # refresh git's credential helper
```

---

## 6. Conventions worth knowing

### 6.1 HTTP / CSRF

- Every state-changing endpoint (POST/PUT/DELETE) MUST call
  `csrf_verify(req.csrf_header, req.body)` and return
  `http_send_csrf_403(client_fd)` on failure. Look at the existing
  `/save-settings`, `/timers/save`, `/api/timers/<n>` handlers for
  the pattern.
- The HTML form pattern is `<input type='hidden' name='csrf' value='%s'>`
  with `csrf_token_hex()` rendered into the page. JSON clients send
  `X-CSRF-Token: <hex>`. Same token, two carriers.
- `req.body` is a fixed 512-byte buffer. For PUT bodies that risk
  exceeding this, plan a different ingestion path — don't silently
  truncate.

### 6.2 Routing

`portal.c` dispatches on `req.path` via a long `if (strcmp(...)) {} else
if (...)` chain. To add a new route:

1. Add the `else if (strcmp(req.path, "/your-path") == 0 && req.method
== REQ_GET)` arm before the `/* ============ 404 ============ */`
   sentinel comment.
2. Render via `snprintf(body + pos, PAGE_BUF_SIZE - pos, ...)`
   accumulator pattern; end with `http_send_page(client_fd, body, pos)`
   for HTML pages or `http_response_start + http_send_body` for JSON.
3. Defensive cap: `if (pos > PAGE_BUF_SIZE - 1024) break;` in loops
   that emit per-element output.

### 6.3 JSON serialization

No external JSON library. Output uses `snprintf` with literal format
strings. **Always escape strings that originated from user input** —
see the `ESC_TO` macro in the `/api/timers` GET handler for the
canonical escape (handles `"`, `\`, and `<0x20` control chars). Don't
emit unescaped `t.topic` or `t.label` into JSON.

### 6.4 Threading

- `broker_task` owns broker state (clients, subs, retained store) and
  is the only thread that mutates them. WS / scheduler / `$SYS` push
  via thread-safe queues; never reach into broker_task's state directly.
- `timers_task` runs at 1 Hz on its own task. Takes the `s_lock` mutex
  in `timers.c` for any access to `s_slots`.
- Portal HTTP serves on `portal_http_task` pinned to CPU 0. Holds no
  long-running locks — all NVS writes are synchronous but short.

### 6.5 UI / portal

- Tasmota-style dark theme. Inline `<style>` in `http_send_page` is the
  source of truth for base CSS; per-page CSS goes in a scoped `<style>`
  block at the top of that route's render.
- Mobile-first: assume 390 px viewport. Use `@media (min-width:600px)`
  to layer desktop affordances on top.
- **Forms must work without JS.** Every `<script>` is progressive
  enhancement only. If you need a server round-trip to apply a change,
  use a `<form>` with a hidden CSRF input and let the browser POST it.
- Color palette is fixed in the header CSS; don't introduce new colors
  without checking the contrast (WCAG AA on `#1f1f1f` background).

### 6.6 NVS writes

- Use the helpers in `portal.c`: `nvs_settings_get_str/u8/u16/i32` and
  `nvs_settings_set_*`. Don't open `nvs_handle_t` directly for the
  `mqtt_cfg` namespace.
- Other namespaces (`ntp`, `wifi`) have their own owner module —
  route reads/writes through that module.
- A failed `nvs_set_*` is silently logged and the device boots with
  defaults. Don't panic-restart on NVS errors.

---

## 7. Constraints to respect

### 7.1 Flash budget

Track every commit's binary delta. Recent baselines:

| Tag   | `mqtt_broker.bin`  | OTA slot free |
| ----- | ------------------ | ------------- |
| 0.7.2 | ~1.13 MB           | 72%           |
| 0.8.0 | 0x11ec20 (1.17 MB) | 72%           |
| 0.8.1 | 0x11f100 (+1.2 KB) | 72%           |
| 0.8.2 | 0x120520 (+5.0 KB) | 72%           |

OTA partition is 4 MB so there's headroom, but **a single commit
adding > 20 KB needs justification in the commit message**.

### 7.2 OTA compatibility

Each release must be OTA-flashable from the previous. That means:

- NVS schema changes need a one-time migration in the loader.
- A version bump can never require a USB re-flash to recover.
- If you add a new partition or change `partitions.csv`, that's a
  major-version bump and you need a separate migration story.

### 7.3 10-year lifetime

Three rules that all stem from "the device must keep working as
deployed for a decade":

1. **No hard-coded policy that governments can change.** DST rules,
   leap-second tables, regulatory band-edges → keep them in NVS as
   user-editable config (POSIX TZ string is the canonical example).
2. **No reliance on cloud / vendor servers.** SNTP upstreams are
   user-configurable. There are no telemetry pings, no "phone home"
   for updates. OTA is operator-driven.
3. **NVS wear awareness.** A change that writes NVS on every MQTT
   PUBLISH would shred the flash in months. The retained store keeps
   to PSRAM exactly because of this. If you add a frequent NVS write
   path, do the math (NVS ~100k erase cycles per page, wear-levelled
   across the partition) and document it.

### 7.4 No external MQTT library

The broker is custom on purpose. Don't pull in Mosquitto, libmosquitto,
Paho C, etc. — they're either fork-incompatible, GPL, or way too heavy
for an MCU. The only non-IDF dependency in `idf_component.yml` is
`espressif/led_strip`.

---

## 8. Testing & verification

Before tagging a release, the following must pass:

1. **`idf.py build` clean** (no new warnings, just the pre-existing
   `-Woverride-init` base64-table noise).
2. **OTA-deploy succeeds** — the device reboots cleanly on the new
   firmware and `/api/ping` returns within ~12 seconds.
3. **Smoke-test the new surface** via curl or the portal UI. For
   API additions: success path + at least one validation-error path
   - missing-CSRF rejection.
4. **Re-capture screenshots** if any UI rendered differently.
   `PORTAL_URL=... PORTAL_AUTH=... python3 tools/capture_*.py`. Commit
   the PNG diffs.
5. **`make test`** (runs `test_broker.py` + `test_ntp.py`) green
   against the live device. Don't introduce a regression in the
   ~129 assertion baseline.

If the change touches the broker fanout, retained store, or QoS path,
also run a manual stress test via `stress_test.py` and watch
`/api/clients` for the in-flight counter going non-zero.

---

## 9. Gotchas (in priority order)

1. **`gh auth` has two accounts on this dev machine** —
   `spencerkittleson` (personal) and `skittleson` (the repo owner). The
   remote is private under `skittleson`. If `git push` returns
   `Repository not found`, run `gh auth switch --user skittleson &&
gh auth setup-git`.
2. **LSP / IDE warnings about `-mlongcalls`, `-mdisable-hardware-atomics`,
   `-fno-shrink-wrap`** are harmless. Those are Xtensa toolchain flags
   that the host's clang LSP doesn't understand; the real Xtensa
   compiler from ESP-IDF handles them fine.
3. **`<input type='time'>` honors browser locale** — US Chrome shows
   `05:00 PM` even if you label the field "24h". The POST body still
   carries `17:00` in 24-hour, so the server is fine, but **never
   claim the field is 24h on screen**.
4. **POSIX TZ `%Z` does not give you "PDT" / "PST"** — it returns the
   literal TZ name token (e.g. `"UTC"` for `UTC7`). Use `%z` and
   format as `(UTC±HH:MM)` for an unambiguous numeric offset.
5. **Edit tool requires reading the file first** if you've never read
   the lines you're about to touch. If you've read lines 1-50 but try
   to edit line 200, you'll get "Edit outside read range". Read the
   region first.
6. **Markdown linters sometimes touch unrelated files** when you save —
   `git status` may show a CHANGELOG-vX.Y.Z.md modified that you didn't
   intend to. `git checkout -- <file>` to revert before committing.
7. **The base portal CSS sets `button { width:100%; line-height:2.4rem;
font-size:1.2rem }`** — any inline pill / small button needs
   `width:auto !important`, `line-height:1.6 !important` to override.
   See `.tmaster` in `portal.c` for the canonical override block.
8. **`req.path` is stripped of `?query` before dispatch.** Use
   `req.query` (also parsed by `http_parse`) to read query params.
   For REST-style paths (`/api/timers/<n>`), match a prefix with
   `strncmp(req.path, "/api/timers/", 12) == 0` and `atoi(req.path + 12)`.
9. **Pre-existing dirty files** in `main/ntp.c`, `main/ntp.h`, and
   `plan-ntp-server.md` have been in the working tree across multiple
   sessions. They are NOT part of your feature unless you specifically
   touched them — exclude from `git add`.
10. **Don't push without explicit user permission.** The pattern is:
    implement → OTA → verify → tag → wait. The human approves the
    push. (`git push --follow-tags` only when asked.)

---

## 10. Quick reference

| Task                                          | Command                                                          |
| --------------------------------------------- | ---------------------------------------------------------------- |
| Enter IDF env                                 | `source $IDF_PATH/export.sh`                                     |
| Build                                         | `idf.py build`                                                   |
| Deploy via OTA                                | `PORTAL_AUTH=support:dockyard make ota`                          |
| Check live version                            | `curl -s -u support:dockyard $URL/api/status \| jq .firmware`    |
| Capture portal screenshots                    | `PORTAL_URL=... PORTAL_AUTH=... python3 tools/capture_portal.py` |
| Capture /timers screenshots                   | `python3 tools/capture_timers.py` (env vars same as above)       |
| Run integration tests                         | `BROKER_AUTH=... make test`                                      |
| Print current firmware version                | `make fmt-version`                                               |
| Switch gh account                             | `gh auth switch --user skittleson && gh auth setup-git`          |
| Get CSRF token (for scripted POST/PUT/DELETE) | `curl -s -u $AUTH $URL/api/csrf \| jq -r .token`                 |

---
> Source: [skittleson/mqtt_broker_esp](https://github.com/skittleson/mqtt_broker_esp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
