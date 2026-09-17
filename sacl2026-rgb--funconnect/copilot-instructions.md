## funconnect

> **Read this first.** Every agent in this project starts here. This file is the

# AGENTS.md — FunConnect Multi-Agent Directive

**Read this first.** Every agent in this project starts here. This file is the
shared context — who we are, what we're building, what's already proven, and
where your own detailed docs live.

---

## 1. What We're Building

**FunConnect** — a device-to-cloud pipeline for classroom IoT. Students connect
microcontrollers (CyberPi, micro:bit, Musebricks) to a live web dashboard
through a single protocol. No MQTT broker. No Python bridge. No laptop relay.

```
CyberPi → ws:// → Cloudflare Edge (TLS termination) ─┐
micro:bit V2 → USB serial → Browser WebSerial relay ─┤
micro:bit V1.5 → WebHID CMSIS-DAP flash (zero-click) │
micro:bit V1.5 → MSD flash (fallback)                 │
CyberPi → Web Serial esptool flash (zero-click) ─────┤
                                                      ▼
                                              Worker → Durable Object (per-device)
                                                       ├── SQLite buffer (hot)
                                                       ├── Madgwick AHRS (~5ms)
                                                       ├── Alarm flush → D1 (cold, queryable)
                                                       └── WSS broadcast → dashboards
```

One API token sees everything. One protocol. One deploy.

**Live URL:** `https://funconnect-v1.funconnect.workers.dev`
**Account:** `CF_ACCOUNT_ID_PLACEHOLDER` (EMAIL_PLACEHOLDER)
**Zone:** `cyberpi.trade` (`CF_ZONE_ID_PLACEHOLDER`)
**Token:** FunConnect (`CF_TOKEN_PLACEHOLDER`)

---

## 2. The Five-Layer Contract (Invariant)

This is the architecture's spine. Every agent's work plugs into one layer.
Swap a layer without changing anything above or below.

| Layer | What | Owned By |
|---|---|---|
| **Transport** | Device speaks WSS + JSON. DO accepts WebSocket. | Firmware + Edge |
| **Storage** | DO-local SQLite (UPSERT telemetry_buffer) → D1 (batch flush, row-ceiling) | Edge |
| **Models** | Madgwick AHRS, signature classification, disturbance corpus | Researcher |
| **Language** | RAG chatbot, prompt engineering, context selection | Agent AI |
| **Interface** | SPA — auth, catalog, dashboard, chat widget | Beauty |

**Rule:** No layer reaches around the contract. Firmware doesn't touch D1.
Beauty doesn't call Madgwick directly. AI doesn't bypass the DO.

---

## 3. Agent Directory

Every agent has a dedicated directory with its own detailed docs. **Read your
own doc before writing code.** Read another agent's doc only when you need to
understand their interface.

| Agent | Directory | Charter Doc | Status |
|---|---|---|---|
| **Alpha** | (root) | `ALPHA.md` | Active — architecture, coordination, non-negotiables |
| **Firmware** | `Firmware/` | `Firmware/FIRMWARE.md` (547 lines) | CyberPi Phase 1–3 done. micro:bit relay + 8 smoke tests + 4 catalog programs delivered. Automation proven. |
| **Edge** | `Edge/` | `Edge/EDGE.md` (992 lines) | Live. UPSERT + dead-alarm fix deployed. py2hex compiler + micro:bit catalog + DAPLink updater shipped. |
| **Researcher** | `Researcher/` | `Researcher/RESEARCHER.md` (1270 lines) | madgwick.ts delivered. 64-signature corpus stable. |
| **Agent AI** | `AI/` | `AI/AI.md` (338 lines) | Live on Qwen 3.7. Keyed reference designed. |
| **Beauty** | `Beauty/` | `Beauty/BEAUTY.md` | Active — catalog UI + MSD flash deployed. webhid-flash.js + WebhidFlashOverlay built (July 20). CyberPi Web Serial integration pending. |
| **Detective** | — | — | Future — architecture audit |
| **Security** | — | — | Future — attack surface, HMAC design |

### How agents interact

- **Alpha** coordinates — assigns work, resolves cross-agent decisions, enforces non-negotiables.
- **Agents own their layer.** Edge doesn't change firmware code. Beauty doesn't touch the DO.
- **When you need context from another agent's domain**, read their charter doc first, then ask Alpha if you still have questions.
- **When your work changes a shared contract** (wire protocol, D1 schema, REST API shape), tell Alpha so the directive can be updated.

---

## 4. Shared Architecture (What Every Agent Must Know)

### 4.1 Wire Protocol (locked — Phase 1–2)

```
DEVICE → DO:
  hello:  {"type":"hello","device_id":"mbot2-01","ts":<epoch_ms>}
  state:  {"type":"state","device_id":"mbot2-01","telemetry":{...},"health":{...}}
  alert:  {"type":"alert","device_id":"mbot2-01","event":"disturbance",
           "accel_peak":<g>,"omega_peak":<rad/s>,"signature":<0-63>,
           "samples":[[ax,ay,az,gx,gy,gz],...],"ts":<epoch_ms>}

DO → DEVICE:
  welcome:     {"type":"welcome","device_id":"mbot2-01"}
  sync:        {"type":"sync","led":false,"doTs":<epoch_ms>}
  ack (state): {"type":"ack","ref":"state","doTs":<epoch_ms>,
                "alert_depth":<N>,"last_flush_ms":<ts>}
  ack (alert): {"type":"ack","ref":"alert"}
  error:       {"type":"error","message":"..."}
  echo:        {"command":"echo","params":{"text":"..."}}
  exec:        {"command":"exec","code":"..."}
  fs_test:     {"command":"fs_test"}

DO → DASHBOARD:
  state:       {"type":"state","device_id":"...","telemetry":{...}}
  alert:       {"type":"alert","device_id":"...","madgwick_json":"..."}

  Alert replay (dashboard connect): Edge sends oldest-first in two
  phases (alert_buffer → D1 alerts, deduped by do_ms). Beauty
  prepends each message — the last (newest) lands at alerts[0]. Live
  alerts arrive individually, same prepend. CONTRACT: either agent
  changing their ordering must notify the other. Edge §28, Beauty
  app.jsx:783.
```

Phase 3 (not built): `set_led` + HMAC, QoS-1 relay queue.

### 4.2 D1 Schema (funconnect-v1-db, UUID a3a8950d…)

```
hello_log    — device_id, timestamp
telemetry    — device_id, created_at, tilt, vibration, acc_x/y/z,
               gyro_x/y/z, uptime_ms, do_ms, health_*
               INDEX: device_id, created_at
alerts       — device_id, event, accel_peak, omega_peak, signature,
               samples, do_ms, madgwick_json
```

### 4.3 REST API (Edge Worker, auth-gated where noted)

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | `/api/catalog` | no | List programs (7 total: 3 CyberPi .py, 4 micro:bit .hex) |
| GET | `/api/catalog/:id` | no | Download program (.py for CyberPi, .hex for micro:bit) |
| GET | `/api/catalog/:id/meta` | no | Program metadata |
| GET | `/api/devices` | no | Recently active devices |
| GET | `/api/device/:id/status` | no | Device online + D1 history |
| POST | `/api/device/:id/echo` | no | Echo command → device |
| POST | `/api/device/:id/exec` | no | Run Python on device |
| POST | `/api/device/:id/fs-test` | no | Filesystem probe |
| POST | `/api/auth/login` | no | admin/admin123 → JWT |
| GET | `/api/me` | JWT | Current user |
| GET | `/api/admin/devices` | JWT | All devices (admin) |
| POST | `/api/chat` | no | RAG chatbot |
| POST | `/api/build` | no | py2hex compiler — .py → .hex (micro:bit) |
| GET | `/api/microbit/relay.hex` | no | Pre-built relay firmware (1.8 MB universal hex) |
| GET | `/api/microbit/daplink-updater.hex` | no | DAPLink firmware updater (V1: v0253, V2: v0258-beta3) |
| WSS | `/device/:id` | — | Device WebSocket upgrade |
| WSS | `/dashboard/:id` | — | Dashboard WebSocket upgrade |

### 4.4 DO Internals (CyberpiHub, per-device)

- **`ctx.acceptWebSocket(server)`** — never `server.accept()`. Hibernation-ready.
- **Zero `await` in `webSocketMessage()`** — sync `ctx.storage.sql.exec()` only.
- **Alarm (60s):** batch flush to D1 via `_flush_registry`. Row-ceiling at 20K. `finally { setAlarm }` — always reschedule.
- **Madgwick:** synchronous in `webSocketMessage()`, ~5ms V8 for 75 samples.
- **UPSERT telemetry_buffer:** `INSERT OR REPLACE`, WITHOUT ROWID, one row per device.

### 4.5 Device Reality

- **CyberPi:** ESP32-D0WD, CyberPiOS v44.01.016. axTLS — no modern TLS. Uses ws:// port 80, TLS terminated at Cloudflare edge. esptool direct flash PROVEN (July 21, 2026) — write .py to flash sector 0x558000, CyberPiOS preserved. Web Serial browser flash with RTS reset also proven. mBlock upload still works as fallback. No USB REPL. LED + LCD diagnostics.
- **micro:bit:** V1.5 and V2. MicroPython via drag-and-drop .hex (py2hex automated). MSD flash works on all hardware. 4 catalog programs verified. V1.5 (KL26Z): WebHID CMSIS-DAP flash PROVEN (July 20, 2026) — one pairing, zero-click thereafter, ~16s. WebUSB permanently blocked by Chromium bug #1150758. V2 (KL27Z): bidirectional serial relay, WebUSB flash viable. V2 constraints: no JSON library, broken uart.readline(), broken display.show(wait=False), USB enumeration race. BLE flashing (Nordic DFU) feasible for future wireless updates.
- **Musebricks:** Planned.

---

## 5. Non-Negotiables (All Agents)

These are hard rules. Violating any of them breaks the architecture.

| # | Rule | Why |
|---|---|---|
| 1 | `ctx.acceptWebSocket(server)` — never `server.accept()` | Hibernation requires the accept pattern |
| 2 | Zero `await` in `webSocketMessage()` | DO billing is wall-clock — sync only on the hot path |
| 3 | Constructor restores from `getWebSockets()` + `deserializeAttachment()` | Survives hibernation |
| 4 | Alarm: `finally { setAlarm }` — always reschedule | One bad deploy permanently kills the alarm otherwise |
| 5 | Five-layer contract — no layer reaches around another | Swappable hardware, swappable models, swappable UI |
| 6 | Resolve in code, not in the prompt | Deterministic lookups belong in functions, not the LLM context |
| 7 | Incremental bisection — one capability per test | No REPL on device = one thing at a time |
| 8 | CPython-first protocol validation | Prove the protocol on laptop before touching hardware |

---

## 6. Source Tree

```
C:\Projects\FunConnect\
├── AGENTS.md                         ← THIS FILE — read first
├── ALPHA.md                          ← Alpha's session log
├── ALPHA_HANDOFF.md                  ← Alpha's handoff (updated July 21)
│
├── Alpha/
│   ├── SESSION-REPORT.md             ← Full session report (July 20-21 breakthroughs)
│   ├── cyberpi-flash-algorithm.md    ← CyberPi read-modify-write algorithm
│   ├── cyberpi-direct-flash.md       ← Original breakthrough notes
│   ├── cyberpi-smoke.html            ← Working Web Serial browser flash page
│   ├── smoke.html                    ← micro:bit WebHID test page
│   ├── auto-smoke.sh                 ← CLI automated smoke test
│   └── findings/
│       ├── webhid-breakthrough.md
│       ├── webhid-technical-supplement.md
│       └── webusb-v15-findings.md
├── ALPHA.md                          ← Alpha's session log
│
├── Edge/
│   ├── EDGE.md                       ← Edge's charter (34 sections, 992 lines)
│   ├── src/index.ts                  Worker routes, auth, catalog, chat, build
│   ├── src/device-hub.ts             CyberpiHub DO
│   ├── src/auth.ts                   JWT sign/verify
│   ├── src/catalog-data.ts           Catalog definitions (7 programs: 3 CyberPi + 4 micro:bit)
│   ├── src/chat.ts                   RAG chatbot (re-export from AI/)
│   ├── src/madgwick.ts               AHRS fusion (from Researcher)
│   ├── src/py2hex.ts                 TypeScript port of uflash 2.0.0 — .py → .hex compiler (295 lines)
│   ├── src/spa-data.ts               SPA HTML inline import (auto-generated)
│   ├── catalog/                      .py files (hello-world, led-blink, imu-stream, heart-badge, name-tag, emotion-badge, dice)
│   ├── firmware-microbit-universal.hex ← V1+V2 MicroPython template (1.85 MB)
│   ├── firmware-daplink-v1.hex       ← DAPLink firmware (currently KL27Z — needs replacement with KL26Z CDN build)
│   ├── firmware-daplink-v2-beta.hex  ← DAPLink v0258-beta3 for V2 (KL27Z, 267 KB)
│   ├── migrations/                   0001–0009 SQL
│   ├── build.js                      esbuild + SPA inline
│   └── deploy.js                     API multipart + D1 migrations
│
├── Firmware/
│   ├── FIRMWARE.md                   ← Firmware's engineering log (547 lines)
│   ├── ws_client.py                   Production CyberPi firmware (dual-state, disturbance, commands)
│   ├── microbit_relay.py             micro:bit relay firmware (LED matrix, hello, echo, 184ms latency)
│   ├── microbit_relay.hex            Pre-built universal hex (1.87 MB)
│   └── microbit_smoke/               P1–P8 smoke test scripts
│
├── Beauty/
│   ├── BEAUTY.md                     ← Beauty's charter + deploy procedure
│   ├── src/app.jsx                   React SPA source
│   ├── spa/index.html                Compiled SPA (98KB, single-file)
│   ├── build.js                      JSX → HTML compiler
│   └── index.template.html           HTML shell
│
├── Researcher/
│   ├── RESEARCHER.md                 ← Researcher's science layer doc (13 sections)
│   ├── madgwick.ts                   AHRS fusion (217 lines) — canonical source
│   ├── signature-analysis.md         64-signature reference
│   └── signature-map.json            Machine-readable export (v1.0.0)
│
└── AI/
    ├── AI.md                         ← Agent AI's NL layer doc (7 sections)
    ├── chat.ts                       RAG chatbot function
    ├── smoke-chat.mjs                Smoke-test harness
    └── live-qwen.mjs                 Qwen live-test script
```

---

## 7. Current State & Remaining Work

### Proven (Phase 1–2 complete)

### Proven (CyberPi — Phase 1)

- esptool CLI direct flash PROVEN (July 21, 2026) — read sector 0x558000, patch text, write back
- Web Serial browser flash PROVEN — same algorithm, Web Serial + esptool-js + RTS reset
- CyberPiOS preserved through all flash operations — factory partition untouched
- mBlock upload still works after esptool writes
- 3 program colors confirmed (blue, red, green) — byte-accurate patching
- Binary metadata at 0xD1 discovered and preserved — the key to making it work

### Proven (Phase 1–2 complete)
- WSS transport (ws:// port 80, axTLS wall)
- Dual-state telemetry (STILL 30s / ACTIVE 4 Hz)
- Disturbance detection (25 Hz jerk gate, 75-sample ring buffer)
- Remote exec over WSS (echo, exec, fs_test)
- UPSERT telemetry_buffer (1 write/frame, D1 row-ceiling 20K)
- Dead-alarm detection (alert_depth + last_flush_ms)
- Madgwick AHRS (adaptive β, 5ms V8, 6 classes)
- 64-signature corpus (11 literature sources, JSON export)
- Auth (JWT, admin/admin123, 24h expiry)
- Chatbot (RAG, Qwen 3.7, auto-thinking)
- Popup upload flow (Path A, bank-grade)
- Auto-follow device tracking

### Proven (micro:bit — Phase 1)

**All hardware:**
- py2hex firmware compiler (POST /api/build, TypeScript, 5ms compile)
- 4 catalog programs (heart-badge, name-tag, emotion-badge, dice) — .hex download + MSD flash
- Pre-built relay firmware (GET /api/microbit/relay.hex)
- DAPLink firmware updater (GET /api/microbit/daplink-updater.hex)
- VID/PID auto-detection (0x0D28: micro:bit, 0x1A86: CyberPi)
- Progressive timeout with firmware download prompts
- Developer automation: py2hex + cp to D:\ (zero human clicks)

**V2 (KL27Z) only:**
- USB serial relay (Web Serial → WSS → DO, same wire protocol)
- LED matrix control via serial commands (5 patterns, 184ms latency)
- Serial read/validate in dev loop
- WebHID CMSIS-DAP flash PROVEN on V1.5 (July 20, 2026) — one pairing, zero-click, ~16s. Replaces WebUSB path which is permanently blocked by Chrome bug #1150758.

**V1.5 (KL26Z) — WebHID flash + MSD fallback.** WebHID CMSIS-DAP zero-click flash proven (July 20). MSD saveToMicrobit is the fallback for non-Chrome browsers. CDC serial is transmit-only — no serial relay.

### Remaining (priority order)

| What | Who | Priority |
|---|---|---|
| ~~Keyed reference lookup (in-code)~~ | ~~Agent AI~~ | ✅ Shipped 2026-07-14 |
| ~~Multi-device support (micro:bit)~~ | ~~All~~ | ✅ Shipped 2026-07-17 |
| Integrate WebHID flash into saveHexToMicrobit cascade (already built in webhid-flash.js, needs wiring) | Beauty | High |
| Integrate Web Serial esptool flash for CyberPi (cyberpi-smoke.html reference in Alpha/) | Beauty | High |
| Fix Edge DAPLink firmware files — mislabeled, need official CDN builds with KL26Z variant | Edge | Medium |
| UI tidying — catalog after connect, V1.5 vs V2 routing | Beauty | Medium |
| Phase 3 set_led + HMAC | Firmware + Edge | Medium |
| micro:bit BLE flashing (Nordic DFU) | Firmware + Beauty | Medium |
| D1 signatures table + admin CRUD | Edge + Researcher | Medium |
| FTS5 documents table + catalog paired docs | Edge + Beauty | Low |
| Multi-tenancy hardening | Edge | Low |
| Conversational mode (Vectorize) | Edge + AI | Deferred |
| Agent Detective — architecture audit | Future | Future |
| Agent Security — attack surface review | Future | Future |

---

## 9. Inter-Agent Communication Protocol

Alpha is the architect. The human operator is the bus — all communication
flows through them. Agents do not message each other directly.

### 9.1 Assignment Format

When Alpha assigns work to an agent, the assignment arrives as a formatted
message from the operator. Every assignment follows this structure:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FROM:     Alpha
TO:       <YourAgent>
SUBJECT:  <one-line summary>
PRIORITY: High | Medium | Low
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

<brief context — what's changed since last session>

TASK:
<specific, verifiable deliverable>

DO NOT:
<things you must not touch, change, or break>

CONTRACT:
<which layer boundary matters, what must remain stable>

REFERENCE:
<files to read before starting, charter sections, handoff docs>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 9.2 Receipt Protocol

When you receive an assignment:

1. **Confirm receipt.** Restate the task in your own words — this proves you
   understood it. Be specific. "I will modify device-hub.ts to add the KL26Z
   DAPLink path" is good. "Got it" is not.
2. **Read the REFERENCE files.** Before writing a single line of code, read
   every referenced file. Know what's already built.
3. **Ask clarifying questions.** If anything in the assignment is ambiguous,
   ask the operator to relay the question to Alpha. Do not guess.
4. **Work within your layer.** The five-layer contract (§2) is invariant. Do
   not reach into another agent's domain.

### 9.3 Handoff Protocol

When you finish work:

1. **Report what you did.** File paths changed, lines added/removed, what was
   proven, what was smoke-tested.
2. **Report what you didn't do.** Tasks in the assignment you deferred,
   and why.
3. **Flag surprises.** Anything you discovered that Alpha should know — bugs,
   platform constraints, quota impacts, integration risks.
4. **Update your charter doc.** Your `AGENT.md` must reflect the new state.
   Update the status header, file manifest, and any stale claims.
5. **Tell the operator.** The operator relays your handoff to Alpha.

### 9.4 Directory Convention

Every agent's domain lives at `C:\Projects\FunConnect\<Agent>\`:

| Agent | Directory | Charter Doc |
|---|---|---|
| Alpha | `C:\Projects\FunConnect\` | `ALPHA.md` |
| Edge | `C:\Projects\FunConnect\Edge\` | `Edge\EDGE.md` |
| Firmware | `C:\Projects\FunConnect\Firmware\` | `Firmware\FIRMWARE.md` |
| Beauty | `C:\Projects\FunConnect\Beauty\` | `Beauty\BEAUTY.md` |
| Researcher | `C:\Projects\FunConnect\Researcher\` | `Researcher\RESEARCHER.md` |
| Agent AI | `C:\Projects\FunConnect\AI\` | `AI\AI.md` |

Your charter is the canonical record of your domain. Read it first every
session. Update it when you ship.

### 9.5 Who Speaks to Whom

```
Alpha ──(through operator)──→ Edge
Alpha ──(through operator)──→ Firmware
Alpha ──(through operator)──→ Beauty
Alpha ──(through operator)──→ Researcher
Alpha ──(through operator)──→ Agent AI

Operator ←──(handoff)── Edge
Operator ←──(handoff)── Firmware
Operator ←──(handoff)── Beauty
Operator ←──(handoff)── Researcher
Operator ←──(handoff)── Agent AI
```

Alpha assigns. Agents deliver. The operator relays everything. No agent
contacts another agent directly. No agent waits for another agent to
finish — Alpha sequences the work.

---

## 10. How to Start — For Any Agent

1. **Read this file first.** You just did.
2. **Check §9.2 Receipt Protocol.** If this session begins with an assignment
   from Alpha, confirm receipt before doing anything else.
3. **Read your own charter doc** (listed in §3 above). It has your domain's
   full history, decisions, and current state.
4. **Read the source files you own** (listed in §6). Know what's already built.
5. **Check ALPHA.md** for the latest architectural decisions and session log.
6. **When you're ready to act**, tell the operator what you're doing. The
   operator relays to Alpha. Alpha coordinates.

Do not read another agent's charter or source unless you need to integrate with
their layer. Each agent's doc is self-contained for its domain.

---

*Agent Alpha — July 21, 2026*

---

## 10. Agent Infrastructure (July 18, 2026)

Tools available to Alpha and all agents through the operator's Reasonix session:

| Tool | Surface | Use |
|---|---|---|
| **Firecrawl MCP** | 26 tools — search, scrape, crawl, map, extract, agent, interact, parse | Web research, content extraction, live page interaction |
| **Cloudflare MCP** | 89 tools — Workers, D1, KV, R2, DO, Queues, AI, Analytics, Zones, Routes | Full Cloudflare platform management (needs session restart) |
| **Cloudflare curl** | REST API via bearer token (CF_TOKEN_PLACEHOLDER) | Same surface as MCP, available immediately |
| **Web Search** | DDG, Wikipedia, Jina, GitHub, Wayback, PubMed | Free web search, no auth required |

The Cloudflare API token (CF_TOKEN_PLACEHOLDER) has full account scope: Workers Scripts Edit, D1 Edit, KV Edit, R2 Edit, Workers AI Read, Account Analytics Read, DNS Edit/Read, Zone Settings Read, SSL Read. One token sees everything.

---
> Source: [sacl2026-rgb/FunConnect](https://github.com/sacl2026-rgb/FunConnect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
