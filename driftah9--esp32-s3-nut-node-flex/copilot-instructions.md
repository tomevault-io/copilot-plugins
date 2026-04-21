## esp32-s3-nut-node-flex

> > This file is read by Claude Code CLI when opening this project.

# Project Rules -- esp32-s3-nut-node-flex
# Type: ESP32 / IDF Firmware (Fork)

> This file is read by Claude Code CLI when opening this project.
> Read after the global CLAUDE.md at /home/claude\.claude\CLAUDE.md

---

## Project Identity

| Field | Value |
|-------|-------|
| Project Name | esp32-s3-nut-node-flex |
| Parent Project | esp32-s3-nut-node (stable main, do NOT backport to it) |
| Forked From | esp32-s3-nut-node v15.18 |
| Target Device | ESP32-S3 (Hosyond ESP32-S3-WROOM-1 N16R8 -- 16MB flash, 8MB PSRAM) |
| IDF Version | v5.3.1 |
| COM Port | COM3 |
| Flash Size | 16MB |
| GitHub Repo | https://github.com/Driftah9/esp32-s3-nut-node-flex |

---

## What This Project Is

Fork of esp32-s3-nut-node investigating tri-mode operation and NUT-style dynamic
HID scanning. The main project is unaffected by anything done here.

### Three operating modes (user selects via web portal, stored in NVS):

**Mode 1 -- STANDALONE (default)**
ESP decodes HID, serves NUT on tcp/3493. No external dependency.
Identical to esp32-s3-nut-node behavior. Works without any infrastructure.

**Mode 2 -- NUT CLIENT**
ESP decodes HID locally, pushes data upstream to a NUT server (upsd).
ESP is a data source, not a server.
Requires: upstream_host + upstream_port in NVS.
Fallback: reverts to Mode 1 if upstream unreachable (configurable).

**Mode 3 -- BRIDGE**
ESP forwards raw USB HID stream over TCP. Zero local decoding.
Sends HID Report Descriptor on connect as handshake, then streams
interrupt-IN packets and GET_REPORT responses raw to upstream host.
Upstream runs NUT with libusb and handles all decode.
Requires: upstream_host + upstream_port in NVS.
Fallback: reverts to Mode 1 if upstream unreachable (configurable).

### Dynamic scanning investigation goals:
- Replace static pre-seeded XCHK rid list with live interrupt-IN accumulation
- Targeted GET_REPORT probe on descriptor-declared rids only (not full sweep)
- Evaluate NUT mge-hid.c mapping table pattern for ESP portability

---

## Source Location

Active source lives at:
```
/home/claude\Projects\esp32-s3-nut-node-flex\src\current\
```
All IDF commands must be run from this directory.

> NOTE: v15.18 baseline has been copied into src\current\. Initial commit pending.

---

## Build / Flash / Monitor Workflow

> BUILD is now CLI-driven via idf-build.ps1.
> FLASH remains manual (COM port access).

### 1 -- Build (CLI runs this)
```powershell
powershell -ExecutionPolicy Bypass -File .\idf-build.ps1 -Target build
```
Output goes to docs\build.log. Claude reads it, fixes errors, rebuilds until clean.

### 2 -- Flash (MANUAL -- Stryder runs this)
```
idf.py flash -p COM3
```
Claude CLI: confirm build clean first, then tell Stryder to flash.

### 3 -- Monitor (MANUAL -- Stryder runs this, CLI reads log)
Stryder saves monitor output to docs\monitor.log.
Claude CLI reads docs\monitor.log after Stryder confirms done.

### 4 -- Update State
After build/flash/monitor confirmed, update docs\project_state.md.

---

## Session Startup Checklist

When Claude CLI opens this project, do these steps in order:

1. Read this file (done)
2. Read docs\project_state.md
3. Read docs\DECISIONS.md
4. Read docs\next_steps.md
5. Note current baseline version and which modules have been modified
6. Proceed with next_steps.md priorities

---

## Upstream Sync Rules

This fork tracks esp32-s3-nut-node as its upstream. When the main project is updated,
changes may be selectively merged into this fork.

**Read-only access to main project source:**
/home/claude\Projects\esp32-s3-nut-node\src\current\

Claude may read any file there at any time for cross-reference.
Never write to, modify, or delete anything in that path.

**When Stryder says "sync from main" or "merge upstream changes":**

1. Read the changed files in esp32-s3-nut-node\src\current\
2. For each changed module, classify it:
   - UNMODIFIED BASELINE: module in flex is identical to original v15.18 baseline
     - Safe to merge upstream changes directly
   - FLEX-MODIFIED: module in flex has been changed for tri-mode or dynamic scanning
     - Must do manual diff review - do NOT auto-apply
     - Present the conflict to Stryder and wait for direction
3. After merging, update README.md to note new upstream baseline version
4. Update docs\DECISIONS.md if the merge affects any architectural decision
5. Use change type SYNC in DOC-REGISTRY.md for the push

**What is safe to always merge without review:**
- ups_device_db.c/h -- new UPS device entries (additive, no conflict risk)
- ups_db_*.c/h -- new device-specific decode files (additive)
- sdkconfig.defaults -- only if flex has not modified partition or USB host settings

**What always requires manual review before merging:**
- cfg_store.c/h -- flex adds op_mode, upstream_host, upstream_port fields
- http_portal.c/h / http_config_page.c/h -- flex adds mode selector UI
- main.c -- flex adds mode dispatch logic
- nut_server.c/h -- may conflict if upstream changes NUT serving logic
- ups_usb_hid.c/h -- may conflict if dynamic scanning changes are in progress

**Baseline tracking:**
Update the "Forked From" line in this CLAUDE.md table whenever a sync is done.
Example: change "esp32-s3-nut-node v15.18" to "esp32-s3-nut-node v15.22" after syncing v15.22.

---

## Versioning

- This fork starts at v0.1 (scaffold)
- Version format: v0.x during investigation phase, v1.x when first mode is stable
- Every source file gets a version comment block at top
- docs\github_push.md is the single source of truth for every push

---

## Git

- Use git-push.ps1 at project root for all GitHub pushes
- Before pushing, ensure docs\github_push.md is current
- Claude updates docs\github_push.md automatically whenever code changes
- Repo: https://github.com/Driftah9/esp32-s3-nut-node-flex

---

## Key Documents

| File | Purpose |
|------|---------|
| docs\project_state.md | Current implementation status |
| docs\DECISIONS.md | Architectural decisions (D001-D004 and growing) |
| docs\next_steps.md | Phased implementation plan |
| docs\github_push.md | Push reference, commit message |

---

## Hard Rules (inherited from main project)

**Em dashes -- NEVER use em dash character (--) in any file or commit message.**
Em dashes corrupt to garbage in PowerShell. Use plain hyphen - or space-hyphen-space instead.
This rule applies to: commit messages, .md files, source comments, config files.

**Shell -- ALL commands in PowerShell or ESP-IDF PowerShell shell. No exceptions.**
- Never use bash, MinGW, MSYS, Git Bash, cmd.exe
- Path separators use backslash in PowerShell commands
- No bash utilities (grep, find, cat, etc.) -- use PowerShell equivalents

**Never backport experimental changes to esp32-s3-nut-node.**
That project stays on known-good static decode paths.

---

## GitHub Identity

- GitHub username: Driftah9
- Repo: https://github.com/Driftah9/esp32-s3-nut-node-flex

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Driftah9) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
