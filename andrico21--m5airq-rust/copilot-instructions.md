## m5airq-rust

> > This file captures hard-won lessons, invariants, and conventions discovered

# CLAUDE.md -- Project Intelligence for M5AirQualityRust

> This file captures hard-won lessons, invariants, and conventions discovered
> during development. AI assistants should read this before making changes.

## Critical Memory Model (ESP32-S3)

**The single most common source of crashes in this project.**

- Total DRAM budget: ~279 KB (BSS + stack + heap all contiguous, zero gap).
- Heap is a `static [u8; N]` array in BSS. Every KB of heap costs 1 KB of stack.
- TLS handshake call chain (WiFi ROM + mbedTLS) needs **≥56 KB stack**. Below that → immediate MWDT1 watchdog crash.
- Current balance: **heap = 88 KB**, **HISTORY_LEN = 360** (6 h), **stack ≈ 82 KB** (22 KB margin).

### Before changing ANY of these values:

1. Build in `--release`.
2. Run: `python scripts/fw_tool.py size`
3. Verify output shows `SIZE OK` and stack ≥ 60 KB.
4. Never reduce HISTORY_LEN below 360 (user requires ≥ 6 hours of history).
5. Never increase heap without verifying stack headroom.

### Memory budget constants (single source of truth):

| Constant | File | Value | Meaning |
|----------|------|-------|---------|
| `heap_allocator!(size: 88 * 1024)` | main.rs | 88 KB | Heap size |
| `HISTORY_LEN` | state.rs | 360 | Samples in ring buffer (6 h @ 1/min) |
| `MBEDTLS_SSL_IN_CONTENT_LEN` | mbedtls_config.h (cargo checkout) | 4096 | TLS read buffer |
| `MBEDTLS_SSL_OUT_CONTENT_LEN` | mbedtls_config.h (cargo checkout) | 4096 | TLS write buffer |

## TLS Singleton Architecture

HTTPS server and MQTTS client share a single `mbedtls_rs::Tls<'static>` instance.
They cannot run simultaneously. Coordination uses two atomic flags:

- `TLS_CLIENT_NEEDED` — MQTTS sets this to request TLS ownership.
- `TLS_CLIENT_READY` — HTTPS signals when it has yielded TLS.

**Critical sequencing**: MQTTS must acquire TLS **before** TCP connect (not after),
otherwise the broker times out waiting for the TLS handshake.

Functions: `request_tls_for_client()`, `tls_client_ready()`, `release_tls_from_client()` in webserver.rs.

## Safety Rules (Expanded)

The copilot-instructions.md says "unsafe only in battery/power detection" but the
actual codebase has justified unsafe in several other areas:

| Location | Reason | Justified |
|----------|--------|-----------|
| battery.rs | SAR ADC2, HWCDC register reads | ✓ Per rules |
| core1_state.rs | f32↔u32 transmute, AtomicStr UnsafeCell | ✓ Lock-free cross-core |
| tls.rs | Cert pointer casting, UnsafeCell for TLS session | ✓ Set-once pattern |
| ota.rs | Flash slot fingerprint read | ✓ Integrity check |
| main.rs | `unsafe impl Send/Sync for SendI2cBus` | ✓ Single-owner I2C |
| state.rs | `software_reset()` FFI (in `reboot()`) | ✓ Reboot only |

**Rule**: Keep `unsafe` minimal, well-justified, and documented with a comment.
Do not add new `unsafe` without verifying no safe alternative exists.

## Known Duplication (Resolved)

`software_reset()` FFI is now centralized in `state::reboot() -> !`.
All call sites (webserver.rs, state.rs) use `crate::state::reboot()` instead
of duplicating the `unsafe extern "C"` block.

## Timeout Consistency

All network timeouts should stay consistent unless there's a specific reason to differ:

| Operation | Timeout | Files |
|-----------|---------|-------|
| DNS query | 10 s | ntp.rs, upload.rs |
| TCP socket operations | 10 s | upload.rs |
| TLS handshake | 30 s | upload.rs |
| WiFi STA connect | 30 s | wifi.rs |
| DHCP lease | 15 s | wifi.rs |
| NTP retry | 30 s | ntp.rs |

If changing a DNS timeout, change it in **both** ntp.rs and upload.rs.

## Log Level Conventions

| Level | Use for |
|-------|---------|
| `error!` | Fatal failures, OOM, persistent I2C/sensor failure |
| `warn!` | Recoverable errors, connection drops, TLS failures |
| `info!` | Normal operational milestones (boot, WiFi connect, measurements) |
| `debug!` | Diagnostics, timing measurements, heap snapshots |

**Do not** use `warn!` for normal user actions (button presses, menu navigation).
**Do not** use `info!` for per-cycle diagnostics (use `debug!`).

## Version String

`FW_VERSION` is declared once in `main.rs` and referenced via `crate::FW_VERSION`
everywhere. Never hardcode version strings.

## Build & Flash

- Always `--release` (no debug builds — LTO and opt-level=s catch real issues).
- Always `--erase-parts otadata` when USB flashing (resets OTA boot selection).
- Port: **COM4** only.
- Pipe build output through `scripts/fw_tool.py build` for clean error extraction.
- mbedTLS compiles from source (not pre-compiled libs) — see build.instructions.md.
- After `cargo update` on mbedtls-rs: reapply SHA nohook patches and SSL buffer size config.

## Common Mistakes (Learned the Hard Way)

1. **Wrapping large async functions in `with_timeout()`** inflates BSS by ~8 KB per
   wrapper (Embassy captures the entire future). Use TCP socket timeouts instead.

2. **Increasing heap without checking stack** causes MWDT1 watchdog crashes during
   TLS handshakes. Always verify with `python scripts/fw_tool.py size`.

3. **Changing HISTORY_LEN** has a large BSS impact (~26 bytes per sample × N).
   360 → 720 costs ~9.4 KB of BSS = 9.4 KB less stack.

4. **TLS singleton deadlock**: If HTTPS server holds TLS with keep-alive and MQTTS
   requests it, both block forever. The `TLS_CLIENT_NEEDED` + `TLS_CLIENT_READY`
   flag protocol prevents this — do not simplify it.

5. **`cargo clean -p mbedtls-rs-sys`** does NOT force C source recompilation.
   Must delete `target/.../build/mbedtls-rs-sys-*` directories manually.

6. **PowerShell stderr is not errors**: cargo/espflash write progress to stderr.
   Red text in terminal does not mean failure. Only `error[E` or `error:` matters.

7. **Heap during TLS sessions** drops to 4-7 KB (from ~35 KB steady state).
   Any code that allocates during a TLS session must be heap-aware.

8. **Battery task disabled**: `battery_task()` spawn is commented out in main.rs.
   `battery_percent` defaults to `-1.0` (sentinel) so the low-power shutdown check
   in sensors.rs (`pct >= 0.0 && pct < threshold`) is safely skipped. Without this
   sentinel, `0.0 < 5.0` triggers immediate shutdown. When re-enabling, restore the
   spawn in main.rs and consider reverting the default to `0.0` if desired.

## Module Responsibilities

| Module | Core | Purpose |
|--------|------|---------|
| board.rs | — | Pin definitions and hardware constants |
| main.rs | 0 | Entry point, hardware init, heartbeat, spawns all tasks |
| wifi.rs | 0 | STA connect with AP fallback |
| webserver.rs | 0 | HTTP/HTTPS server (picoserve), TLS singleton owner |
| upload.rs | 0 | MQTT/MQTTS/HTTP telemetry upload |
| ntp.rs | 0 | SNTP time sync |
| ota.rs | 0 | OTA firmware update (dual-slot) |
| tls.rs | 0 | TLS server/client setup, cert management |
| auth.rs | 0 | Digest authentication (RFC 7616) |
| sensors.rs | 1 | SCD40 + SEN55 I2C polling |
| display_ui.rs | 1 | E-ink rendering |
| display/mod.rs | 1 | SSD1681 driver re-exports |
| buttons.rs | 0 | GPIO0/GPIO8 debounced input |
| battery.rs | 1 | ADC2 voltage + HWCDC USB detection (**DISABLED** — task spawn commented out in main.rs) |
| rtc.rs | 1 | PCF8563 RTC read/write |
| state.rs | — | Global shared state, ring buffer, settings access |
| core1_state.rs | — | Lock-free cross-core atomics |
| settings.rs | — | Config schema with defaults |
| flash_store.rs | — | NOR flash KV store |
| time_utils.rs | — | Timezone parsing, epoch formatting |

## File Organization

- `src/` — All Rust source modules.
- `.github/copilot-instructions.md` — Copilot context (loaded automatically).
- `.github/instructions/` — Domain-specific instruction files (build, display, memory, wifi).
- `.github/prompts/` — Reusable prompt templates.
- `partitions.csv` — ESP32 partition table (ota_0, ota_1, nvs).
- `HARDWARE.md` — Pin map, sensor datasheets, power circuit.
- `scripts/fw_tool.py` — Build/flash/serial output filter + DRAM stack headroom checker (`size` subcommand).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andrico21)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andrico21)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
