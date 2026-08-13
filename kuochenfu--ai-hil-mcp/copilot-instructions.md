## ai-hil-mcp

> validates it and shows what each board resolves to.

# CLAUDE.md — AI-HIL Embedded Dev Automation

Operating instructions for Claude Code on this repository.
Follow the SOPs below exactly when working with physical hardware.

---

## Project Overview

**AI-HIL (AI-Hardware-in-the-Loop)** lets Claude Code perceive, act on and
validate physical embedded hardware through MCP servers.

Layout:

```
Cargo.toml          workspace root — one target/ for all six servers
hil-core/           shared: device registry, anomaly rules, resource locks,
                    command execution with timeouts, structured tool output
<name>-mcp-rs/      the six MCP servers
devices.toml        single source of truth for every board, camera and mic
orchestrator/       Python closed-loop driver + metrics
doc/known-bugs.md   the Known Bug Record
```

## Active MCP Servers

| Server | Tools |
|--------|-------|
| `serial-mcp` | `list_serial_ports`, `list_boards`, `read_serial_log`, `send_serial_command`, `read_multi_log`, `wait_for_pattern`, `start_logging`, `stop_logging`, `clear_log_buffer` |
| `jtag-mcp` | `list_probes`, `list_boards`, `halt_cpu`, `resume_cpu`, `reset_target`, `read_registers`, `read_memory`, `write_memory`, `read_call_stack`, `diagnose_hardfault`, `set_breakpoint`, `clear_breakpoint`, `set_watchpoint`, `clear_watchpoint`, `read_rtt_log`, `start_rtt_logging`, `stop_rtt_logging` |
| `build-flash-mcp` | `list_projects`, `build_firmware`, `clean_build`, `get_build_size`, `flash_firmware`, `erase_flash` |
| `ppk2-mcp` | `find_ppk2`, `get_metadata`, `measure_current`, `profile_power_states`, `measure_with_pin_trigger`, `estimate_battery_life`, `set_dut_power` |
| `vision-mcp` | `list_cameras`, `list_boards`, `get_camera_info`, `set_resolution`, `set_ptz`, `adjust_image`, `capture_frame`, `start_stream`, `stop_stream`, `grab_frame`, `start_recording`, `stop_recording` |
| `audio-mcp` | `list_audio_devices`, `get_audio_info`, `capture_audio`, `detect_frequency`, `measure_noise_level`, `detect_tone` |

Binaries live in `target/release/` (workspace build). Rebuild with `./setup.sh`.

### How to read tool results

Every tool returns **human text plus a structured JSON body**, and failures set
`isError`. Branch on the JSON, not on the prose:

- `ok: true|false` is present on every result.
- `build_firmware` → `built: true|false`; `flash_firmware` → `flashed: true|false`.
- `diagnose_hardfault` → `cfsr_flags: []`, `hfsr_flags: []`, `in_fault: bool`, `fault_pc`.
- `read_serial_log` / `read_rtt_log` → `lines: []`, `anomaly: { kind, matches }`.
- `capture_frame` / `grab_frame` → an actual image block you can look at.

A failed tool call is an error, not text to interpret. If `build_firmware`
returns `isError`, **do not flash** — fix the build first.

## Target Hardware

Everything is declared in `devices.toml`; `cargo run -p hil-core --example doctor`
validates it and shows what each board resolves to.

- **STM32WL55JC** (NUCLEO-WL55JC1 x4) — sub-GHz LoRa, ultra-low-power, dual-core (CM4 + CM0PLUS)
  - Aliases `stm32a`–`stm32d`, each pinned to its own ST-Link by `probe_serial`
  - Firmware: `/Users/chenfu/Labs/stm_projects/synapse-lora`, preset `Debug`
    (the parent project builds both CM4 and CM0PLUS)
  - `artifacts` in `devices.toml` names the two ELFs explicitly — the tree also
    holds stale copies under `build/`, so never rely on globbing
  - RTT logging is available over SWD (`read_rtt_log`), faster than UART and
    usable while the CPU is blocked
  - **Option bytes:** SBRV must be `0xC000` (CM0+ boots from `0x08030000`).
    Factory default `0x8000` points at erased flash → CM0+ never boots → CM4
    hangs at mailbox sync.
- **ESP32-S3 Gateway** (`board1`) and **Node** (`board2`) — Zenoh relay / sensor node
  - Build with `idf`, flash with `esptool`; log and shell are separate ports
- **Nordic PPK2** — power measurement; auto-detected, or set `ppk2_port` per board
- **Logitech MX Brio Ultra 4K** — visual inspection (`camera.brio`, index 0)

---

## Safety Constraints

Some of these are now enforced in code; the rest still depend on you.

**Enforced by the servers:**
- Supply voltage is capped at each board's `max_voltage_mv` (default 3600 mV).
  `ppk2-mcp` refuses anything higher instead of driving it into the DUT.
- `erase_flash` requires `confirm=true`.
- Every probe/port operation takes a cross-process lock, so a flash cannot
  collide with a live debug session or a log collector. A contended call fails
  with the name of the holder rather than corrupting the session.
- External commands (OpenOCD, idf.py, esptool, cmake) run under a timeout.

**Your responsibility:**
- Never modify an ISR without reading the call stack first (`read_call_stack`).
- Never flash if the build returned `isError`.
- Wait 3 s after flashing before reading the log — the board needs to boot.
- Watchdog timeout is 2 s: do not leave the CPU halted for more than ~1.5 s
  during live diagnosis.
- If `diagnose_hardfault` shows `FORCED` in `hfsr_flags`, always read
  `cfsr_flags` for the real cause before touching code.
- ESP32: `resume_cpu` may fail after a halt on Xtensa — use `reset_target`.

### Who owns the hardware

Only one thing can hold a probe or a port at a time:

| Before this | Do this |
|---|---|
| `flash_firmware` on an ESP32 | `stop_logging(ports="board1/log")` — esptool needs the port |
| `flash_firmware` on an STM32 | `stop_rtt_logging(board=...)` if an RTT collector is running |
| Long RTT capture | `start_rtt_logging` — it claims the probe until you stop it |

After flashing an ESP32, call `start_logging` again to capture boot output.

---

## Orchestrator SOP

Execute in order. Do not skip steps, and do not ask for confirmation between
them unless you are blocked.

### Step 1 — Triage

```
read_serial_log(port="stm32a/log", lines=50, timeout_s=8)
```

For STM32, RTT is faster and works while the CPU is blocked:

```
read_rtt_log(board="stm32a", lines=50, timeout_s=8)
```

Use `read_multi_log(ports="stm32a/log,board1/log")` to sample several boards at
once. Read `anomaly.kind` from the result:

| `anomaly.kind` | Go to |
|---|---|
| `hardfault` | Step 2A |
| `panic` | Step 2B |
| `watchdog` | Step 2C |
| `none`, but no output at all | Step 2D |
| `none`, with healthy output | Stop — report "no anomaly detected" |

### Step 2A — HardFault / stack overflow

Run in parallel:

```
diagnose_hardfault(board="stm32a")   ← decodes HFSR/CFSR/BFAR/MMFAR
read_call_stack(board="stm32a")      ← exception frame at SP
read_registers(board="stm32a")       ← register snapshot
```

Interpreting `cfsr_flags`:

- `PRECISERR` + `BFARVALID` → precise bus fault; `bfar` holds the address
- `IACCVIOL` → jumped to an invalid address; check `pc` and `lr`
- `DIVBYZERO` → integer divide by zero; `fault_pc` locates the function
- `STKERR` → stack overflow; compare `sp` against the stack bottom
- `FORCED` in `hfsr_flags` → escalated fault; `cfsr_flags` holds the real cause

`fault_pc` (the PC from the exception frame) is the faulting instruction —
cross-reference it with the map file or ELF symbols.

### Step 2B — Panic / assert

```
read_serial_log(port="stm32a/log", lines=100)
read_registers(board="stm32a")
```

Extract the file, line and condition from the panic output.

### Step 2C — Watchdog reset

```
read_serial_log(port="stm32a/log", lines=100)
read_registers(board="stm32a")
```

Look for long-running loops, blocking waits, or a missing `HAL_IWDG_Refresh()`.

### Step 2D — Board hang (no output)

```
halt_cpu(board="stm32a")
read_registers(board="stm32a")
read_call_stack(board="stm32a")
```

`halt_cpu` marks the core as deliberately halted, and the diagnostic tools then
leave it halted so successive reads describe the same moment. Call `resume_cpu`
when you are done, or pass `resume=true` on the last read.

Check whether the CPU is spinning in a tight loop, blocked on a semaphore, or
looping in a fault handler.

### Step 3 — Remediation

1. Identify the source file and function at fault.
2. Read the relevant sources before changing anything.
3. Apply the minimal fix; do not refactor unrelated code.
4. If touching an ISR, re-read the call stack first and check the fix does not
   affect interrupt timing.

### Step 4 — Build and flash

**STM32 (CMake + ST-Link over SWD — the serial port is unaffected):**

```
build_firmware(board="stm32a")        ← builds CM4 + CM0PLUS
flash_firmware(board="stm32a")
```

All four boards can be flashed in parallel; each targets its own ST-Link:

```
flash_firmware(board="stm32a")
flash_firmware(board="stm32b")
flash_firmware(board="stm32c")
flash_firmware(board="stm32d")
```

**ESP32-S3 (idf.py + esptool — the port must be released):**

```
build_firmware(board="board1")
stop_logging(ports="board1/log")
flash_firmware(board="board1")
start_logging(ports="board1/log")
```

If the build fails, fix the compile errors and go back to Step 3. Do not flash.
Wait 3 seconds after flashing completes.

### Step 5 — Verification

```
read_serial_log(port="stm32a/log", lines=30, timeout_s=5)
```

Boot output is already buffered — the collector keeps running across a flash.

Pass criteria:
- `anomaly.kind` is `none`
- the expected boot banner is present (e.g. `System Initialization`, `Tx PING`)
- the board is producing output at all

On failure, return to Step 2 with the new output as context.

### Step 6 — Record

Append an entry to `doc/known-bugs.md` using the format documented at the top of
that file.

---

## Architecture Reference

```
Brain:           Claude Code CLI + this file
Shared core:     hil-core (registry · anomaly rules · locks · timeouts · output)
Perception:      serial-mcp (UART) · jtag-mcp (registers, faults, RTT)
                 ppk2-mcp (power) · vision-mcp (camera) · audio-mcp (acoustics)
Action:          build-flash-mcp (CMake / idf.py → OpenOCD / esptool / probe-rs)
```

### Closed-loop flow

```
Triage → Diagnosis → Remediation → Build & Flash → Verification → Record
  ▲                                                      │
  └──────────────────── FAIL ────────────────────────────┘
```

### Transports

stdio is the default and is what `.mcp.json` uses. Setting `AI_HIL_HTTP_ADDR`
runs a server over MCP streamable HTTP instead, as a long-lived instance that
owns the hardware and is shared by several clients:

```sh
AI_HIL_HTTP_ADDR=127.0.0.1:8181 target/release/serial-mcp-rs
python orchestrator/orchestrator.py --mcp-url http://127.0.0.1:8181/mcp
```

The orchestrator prefers this path. Without it, it falls back to opening the
port directly under the same lock the servers use.

---

## Known Bug Record

Moved to [`doc/known-bugs.md`](doc/known-bugs.md) so this file stays a fixed
size. Append there in Step 6.

---
> Source: [kuochenfu/ai-hil-mcp](https://github.com/kuochenfu/ai-hil-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
