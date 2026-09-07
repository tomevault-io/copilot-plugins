## dartslive-scorer

> > **CRITICAL INSTRUCTION FOR ALL AI AGENTS**: Read this file completely before inspecting or modifying code in this directory. Before completing your session, you MUST run existing tests and record your changes in the Session Log at the bottom.

# AGENTS.md: dartslive-scorer

> **CRITICAL INSTRUCTION FOR ALL AI AGENTS**: Read this file completely before inspecting or modifying code in this directory. Before completing your session, you MUST run existing tests and record your changes in the Session Log at the bottom.

## 1. Project Identity & Purpose
- **Core Stack**: Node.js, Express 4.18.2 (backend), vanilla HTML/CSS/JS + Web Bluetooth API (frontend)
- **Primary Objective**: Web-based darts scoring app for a DARTSLIVE Home dlb0003 dartboard, connecting via Web Bluetooth. Includes a small Express server for a shared, disk-persisted leaderboard across devices on the LAN, plus optional Home Assistant/WLED LED-ring integration tied to game state.
- **Entry Points**: `server.js` (`npm start` — Express app + static file serving), `ble-scanner.html` (standalone Phase 1 BLE discovery tool), `index.html` / `darts-scorer.html` (main scoring app)

## 2. Architecture & Dependencies
- **Internal Modules**:
  - `server.js` — Express server, `/api/*` routes, serves leaderboard history and triggers HA scripts
  - `scorer.js` — client-side game logic (01 and Cricket)
  - `ble.js` — Web Bluetooth driver + dart segment byte decoder (Nordic UART service `6e400001-...`, char `6e40fff6-...`)
  - `add-game.js` / `remove-player.js` — utility scripts
  - `data/` — `history.json`, persisted leaderboard state (gitignored)
  - `certs/` — `cert.pem` / `key.pem` (do not read/print contents — TLS material)
- **Key External APIs / Services**: Home Assistant REST API (optional — activates WLED presets on a dartboard LED ring via HA scripts); Web Bluetooth (browser-native, no server-side dependency)
- **Environment Variables Required**: `HA_BASE_URL`, `HA_TOKEN`, `HA_SCRIPT_GAME_ON`, `HA_SCRIPT_GAME_OFF`, `HA_SCRIPT_WINNER` (all optional — app runs fine with none set; see `.env.example`)

## 3. Build, Test, & Execution Commands

```bash
# Install dependencies
npm install

# Run local dev/test server
npm start
# Open http://localhost:3180 (or http://<lan-ip>:3180) in Chrome/Chromium

# Execute test suite
# No automated test suite exists in this project.
```

## 4. Current State & Known Technical Debt
 * **Status**: Functional — runs as a persistent Windows service.
 * **Known Technical Debt / Bugs**:
   * [ ] No automated test suite.
   * [ ] The legacy static-file serving path (`python3 -m http.server`) silently drops the shared leaderboard and LED integration (no `/api/*` routes) — falls back to browser `localStorage` with no warning; can look like a data-loss bug if someone serves it this way by habit.
 * **Key Constraints**:
   * The production Windows Service is hosted by **`C:\ClaudeCentral\tools\nssm\nssm.exe`**, a path *outside this repo*. Service config: `AppDirectory=C:\ai\dartslive-scorer`, `Application=C:\Program Files\nodejs\node.exe`, `AppParameters=server.js`. If `C:\ClaudeCentral` is ever cleaned up, that `tools\nssm` subfolder must be preserved (or the service migrated to a new NSSM install) or the service loses its host process.
   * `.env` holds a Home Assistant long-lived access token — never commit it; it's already gitignored.

## 6. Security & Runtime Audit (2026-08-02)
- **Credential sweep (working tree)**: no credential-name literal hits found. `certs/cert.pem` and `certs/key.pem` (TLS material) confirmed gitignored — tracked: no, pushed: no, in-history-only: no.
- **Git history check**: `git log --all --oneline -- .env`, `-- certs/cert.pem`, `-- certs/key.pem` each returned no commits — none were ever tracked. Repo-wide sweep for added files matching `.env|secret|password|credential|\.pem` across all history found only `.env.example` (placeholder template, expected/safe).
- **Runtime status**: A `node` process (PID 15184) listens on ports 3180/3181, matching this project's documented port. Identified as Windows Service **`dartslive-scorer`** (DisplayName "Dartslive Scorer"), state RUNNING, hosted via NSSM (`C:\ClaudeCentral\tools\nssm\nssm.exe`) with `Application=C:\Program Files\nodejs\node.exe`, `AppParameters=server.js`, `AppDirectory=C:\ai\dartslive-scorer` — consistent with the NSSM config already documented in Section 4. Healthy, correctly correlated.
- **Gitignored-but-present directories**: `.env` (378 bytes, not tracked), `certs/` (2.9K, not tracked), `data/` (3.4K, leaderboard history), `node_modules/` (2.3M). No size or content concerns.
- **`C:\ClaudeCentral` cleanup confirmation (2026-08-02, follow-up)**: verified `C:\ClaudeCentral\` currently contains exactly two things — `dartslive-scorer\` (a full 4.4 MB old copy of this project, own `.git`, own `.env`, `node_modules/` — **superseded**, the live NSSM service's `AppDirectory` is confirmed `C:\ai\dartslive-scorer`, not this old copy) and `tools\` (`migrate-stage1.ps1`, `verify-persistence.ps1`, and `tools\nssm\` — **must be kept**, `nssm.exe` here is confirmed to be actively hosting the running `dartslive-scorer` Windows service right now, `nssm.exe status dartslive-scorer` returned `SERVICE_RUNNING`). Net: `C:\ClaudeCentral\dartslive-scorer\` is safe to remove once someone's satisfied nothing else references it (its own `.env` should be checked/rotated-if-needed before deletion, same as any other credential file, rather than assumed harmless because it's "just a duplicate"); `C:\ClaudeCentral\tools\nssm\` must NOT be removed — the live service depends on it.

## 7. Session Log (Mandatory Agent Updates)
*Format: [YYYY-MM-DD] [Agent Name/Model] - Summary of changes made*
 * [2026-08-02] [Setup-Agent] - Migrated legacy benchmark/context docs to consolidated AGENTS.md format.
 * [2026-08-02] [Audit-Agent] - Ran security/runtime audit: confirmed `.env`/certs never committed (working tree + full git history), correlated the NSSM-hosted `dartslive-scorer` service with the process on ports 3180/3181, reviewed gitignored dirs. No issues found; see Section 6.

---
> Source: [drfletcher2k/dartslive-scorer](https://github.com/drfletcher2k/dartslive-scorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
