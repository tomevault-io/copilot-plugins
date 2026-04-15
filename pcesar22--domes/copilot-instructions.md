## domes

> After ANY code implementation, verify changes work END-TO-END. Unit tests alone are NOT sufficient.

# DOMES Project — Claude Code Instructions

## MANDATORY: Verification After Implementation

After ANY code implementation, verify changes work END-TO-END. Unit tests alone are NOT sufficient.

### Verification Levels (ALL required for new features)

#### Level 1: Unit Tests
```bash
cd firmware/test_app && mkdir -p build && cd build && cmake .. && make && ctest --output-on-failure
```

#### Level 2: Build ALL affected components
```bash
# Firmware
cd firmware/domes && . ~/esp/esp-idf/export.sh && idf.py build

# Host tools (if modified)
cd tools/domes-cli && cargo build
```

#### Level 3: End-to-End Verification (REQUIRED for new features)
```bash
/flash
# Then test the actual feature via CLI, serial output, or visual confirmation
```

### Feature-Specific Verification

| Feature Type | Verification Method |
|--------------|---------------------|
| Protocol/transport | Flash, run host tool, verify communication |
| Config/runtime | Flash, use CLI to change settings, verify applied |
| LED/display | Flash, visually confirm behavior |
| Sensors/input | Flash, trigger input, verify response in logs |
| WiFi transport | Flash, run `domes-cli --wifi <IP>:5000 feature list` |
| Serial transport | Flash, run `domes-cli --port <PORT> feature list` |
| BLE transport | Flash, run `domes-cli --ble "DOMES-Pod-XX" feature list` |
| Multi-device | Flash all, run `domes-cli --all feature list` |
| ESP-NOW | Flash both, enable esp-now, monitor for peer discovery |
| OTA updates | Flash, run `domes-cli ota flash`, verify reboot |
| **CI verification** | Add `hw-test` label to PR |

### Available Skills & Commands

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `/flash` | Build, flash, monitor | **ALWAYS** after firmware changes |
| `/lint-fw` | Check coding standards | Before committing firmware code |
| `/size` | Analyze binary size | After adding new features |
| `/debug-esp32` | GDB debugging | When investigating runtime issues |

**DO NOT** mark a task as complete if build fails, tests fail, firmware doesn't flash, or the feature hasn't been verified on hardware.

---

## MANDATORY: Use Git Worktrees for Features

```bash
mkdir -p .worktrees
git worktree add .worktrees/<name> -b claude/<type>/<description>
cd .worktrees/<name>
# Types: feat, fix, refactor, docs
```

- Never work directly on `main` for features/fixes
- **Always ask** before creating a PR
- Clean up: `git worktree remove .worktrees/<name>`

---

## CRITICAL: Protocol Buffers (nanopb/prost)

**ALL message serialization between firmware and host tools MUST use Protocol Buffers.**

```
┌─────────────────────────────────────────────────────────────────┐
│  NEVER HAND-ROLL PROTOCOL DEFINITIONS.                         │
│  NEVER DUPLICATE ENUMS OR MESSAGE TYPES IN CODE.               │
│  ALL DEFINITIONS COME FROM .proto FILES.                       │
│  IF YOU CREATE A NEW MESSAGE TYPE, ADD IT TO THE .proto FILE.  │
└─────────────────────────────────────────────────────────────────┘
```

- **Firmware**: nanopb (C library for embedded)
- **Host CLI (Rust)**: prost (Rust protobuf library)
- **Single source of truth**: `firmware/common/proto/*.proto`

```
config.proto (SOURCE OF TRUTH)
       │
       ├──► nanopb generator ──► config.pb.h / config.pb.c (firmware)
       └──► prost build.rs ──► config.rs (CLI, generated at build time)
```

### Adding a New Feature/Message

1. Edit `firmware/common/proto/config.proto`
2. Rebuild firmware (`idf.py build` — nanopb runs automatically)
3. Rebuild CLI (`cargo build` — prost runs automatically via build.rs)
4. Use the generated types in your code. **NEVER** manually define enums in C++ or Rust.

### Key Proto Files

| File | Purpose |
|------|---------|
| `firmware/common/proto/config.proto` | **SOURCE OF TRUTH** for all protocol definitions |
| `firmware/common/proto/config.options` | nanopb size constraints |
| `tools/domes-cli/build.rs` | prost code generation for CLI |
| `tools/domes-cli/src/proto/` | **Generated — do not edit** |

---

## Runtime Configuration Protocol

Binary protocol over USB serial, WiFi TCP, or BLE GATT.

- **Frame format**: `[0xAA][0x55][LenLE16][Type][Payload][CRC32LE]`
- **Payload**: Protobuf-encoded messages
- **Transports**: USB-CDC (serial), TCP port 5000 (WiFi), BLE GATT (notifications)

### Available Features (defined in config.proto)

| ID | Feature | Description |
|----|---------|-------------|
| 1 | FEATURE_LED_EFFECTS | LED animations and effects |
| 2 | FEATURE_BLE_ADVERTISING | Bluetooth Low Energy advertising |
| 3 | FEATURE_WIFI | WiFi connectivity |
| 4 | FEATURE_ESP_NOW | ESP-NOW peer-to-peer |
| 5 | FEATURE_TOUCH | Touch sensing |
| 6 | FEATURE_HAPTIC | Haptic feedback |
| 7 | FEATURE_AUDIO | Audio output |

### Testing Config Protocol

```bash
# Serial
domes-cli --port /dev/ttyACM0 feature list

# WiFi
domes-cli --wifi <ESP32_IP>:5000 feature list

# BLE (native Linux only)
domes-cli --scan-ble
domes-cli --ble "DOMES-Pod-01" feature list
```

### Key Files

| File | Purpose |
|------|---------|
| `firmware/domes/main/config/configCommandHandler.hpp` | Command handler (serial/TCP/BLE) |
| `firmware/domes/main/config/featureManager.hpp` | Feature state management |
| `firmware/domes/main/transport/bleOtaService.hpp` | BLE GATT service (NimBLE) |
| `tools/domes-cli/src/transport/ble.rs` | CLI BLE transport (btleplug) |

---

## OTA Firmware Updates

The `domes-cli` tool is the **only supported method** for OTA updates. Supports serial, WiFi/TCP, and BLE.

```bash
# Serial OTA
domes-cli --port /dev/ttyACM0 ota flash firmware/domes/build/domes.bin --version v1.0.0

# WiFi OTA
domes-cli --wifi 192.168.1.100:5000 ota flash firmware/domes/build/domes.bin

# Multi-device OTA
domes-cli --all ota flash firmware/domes/build/domes.bin --version v1.0.0
```

### OTA Protocol Flow

```
Host → Device: OTA_BEGIN (size, sha256, version)
Device → Host: OTA_ACK (status=OK, nextOffset=0)
Host → Device: OTA_DATA (offset=0, data[0..1015])  ← repeat for all chunks
Device → Host: OTA_ACK (status=OK, nextOffset=1016)
Host → Device: OTA_END
Device → Host: OTA_ACK (status=OK) → device reboots
```

### Key Files

| File | Purpose |
|------|---------|
| `tools/domes-cli/src/commands/ota.rs` | CLI OTA implementation |
| `firmware/common/protocol/otaProtocol.hpp` | OTA message definitions |
| `firmware/domes/main/transport/serialOtaReceiver.hpp` | Device-side OTA handler |
| `firmware/domes/main/services/otaManager.hpp` | HTTPS/GitHub OTA (auto-updates) |

---

## Performance Tracing

Lightweight tracing framework with Perfetto export.

```bash
# Dump traces
python tools/trace/trace_dump.py -p /dev/ttyACM0 -o trace.json -n tools/trace/trace_names.json

# Multi-device
python tools/trace/trace_dump.py --ports /dev/ttyACM0,/dev/ttyACM1 -o trace.json

# Visualize at https://ui.perfetto.dev
```

```cpp
#include "trace/traceApi.hpp"

void myFunction() {
    TRACE_SCOPE(TRACE_ID("MyModule.Function"), domes::trace::TraceCategory::kGame);
    TRACE_INSTANT(TRACE_ID("MyModule.Event"), domes::trace::TraceCategory::kGame);
    TRACE_COUNTER(TRACE_ID("MyModule.Value"), someValue, domes::trace::TraceCategory::kGame);
}
```

Full documentation: `research/architecture/07-debugging.md`

---

## Multi-Device Testing

Two NFF devboards connected via USB as `/dev/ttyACM0` and `/dev/ttyACM1`.

### Device Registry

```bash
domes-cli devices add pod1 serial /dev/ttyACM0
domes-cli devices add pod2 serial /dev/ttyACM1
domes-cli devices list
domes-cli devices scan  # auto-discover serial + BLE
```

### Common Operations

```bash
domes-cli --all feature list
domes-cli --all led solid --color ff0000
domes-cli --all ota flash firmware/domes/build/domes.bin
domes-cli --target pod1 --target pod2 wifi status
```

### BLE Device Naming

- `DOMES-Pod-01` (pod_id = 1)
- `DOMES-Pod-02` (pod_id = 2)
- `DOMES-Pod-3A2B` (fallback: last 2 bytes of BT MAC)

### ESP-NOW Testing

```bash
domes-cli --all feature enable esp-now
python .claude/skills/esp32-firmware/scripts/monitor_serial.py /dev/ttyACM0,/dev/ttyACM1 30
```

---

## Platform Requirements

**Full details: `.claude/PLATFORM.md`**

- Host: **Native Linux** (Arch, Ubuntu) — BLE works directly
- BLE adapter: Intel AX200/AX210 or Realtek RTL8761B
- Do NOT use WSL2 for BLE (USB passthrough broken)
- Do NOT use Raspberry Pi (BCM Bluetooth firmware bugs)

---

## CI/CD Infrastructure

The `hw-test` label on a PR triggers queued hardware testing on a self-hosted runner.

**What CI tests:** build firmware → build CLI → flash → feature list → Serial OTA → post-OTA verify → multi-device (if 2+ devices)

```bash
# Trigger hardware CI
gh pr edit --add-label "hw-test"
```

Runner requirements: ESP-IDF v5.2.2+, Rust toolchain, ESP32 device(s), `dialout` group.

---

## Learnings & Gotchas

### Initialization Order Matters

In `main.cpp`, components must be initialized in this order:
1. **WiFi** before TCP config server and BLE (for coexistence)
2. **BLE OTA service** early (advertising starts automatically)
3. **FeatureManager** before TCP/Serial/BLE config handlers
4. **TCP config server** before Serial OTA (to see logs)

**BLE Note**: GATT characteristic handle only valid after NimBLE syncs. Uses global `g_statusCharHandle` populated after sync.

### USB-CDC Console Takeover

After `initSerialOta()`, USB-CDC becomes the OTA/config transport and logs stop on serial monitor. To debug init issues: add delays before serial OTA init, or temporarily disable it.

### Always Search Codebase First

Before asking the user, search:
```bash
grep -r "kWifiSsid\|kWifiPassword" firmware/  # WiFi credentials
grep -r "CONFIG_DOMES" firmware/domes/sdkconfig*  # Feature flags
```

---

## Documentation Map

| Document | Purpose |
|----------|---------|
| `firmware/CLAUDE.md` | Coding standards, architecture rules, firmware-specific guidance |
| `firmware/MILESTONES.md` | Development phases and current status |
| `research/SYSTEM_ARCHITECTURE.md` | Hardware architecture and specs |
| `research/architecture/` | Deep design docs (00-12) |
| `docs/PIN_REFERENCE.md` | GPIO pin mappings for all platforms |
| `tools/domes-cli/CLAUDE.md` | CLI tool guidance |
| `hardware/CLAUDE.md` | Hardware component sourcing |
| `.claude/PLATFORM.md` | Platform requirements, BLE setup, udev rules |
| `.claude/skills/` | Debug, firmware, and workflow skills |

---

*Last updated: 2026-03-22*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pcesar22) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
