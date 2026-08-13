## tailfirmware

> Firmware for ESP32-C3 / ESP32-S3 that controls a 2-axis animatronic tail with motion patterns, LED effects, IMU sensing, and BLE configuration.

# TailFirmware

Firmware for ESP32-C3 / ESP32-S3 that controls a 2-axis animatronic tail with motion patterns, LED effects, IMU sensing, and BLE configuration.

## Build

Requires: ESP-IDF v5.4.1 (installed via Espressif Windows installer at `C:\Espressif`).
Both the RISC-V (C3) and Xtensa (S3) toolchains must be installed; `build.bat`
puts both on PATH.

```bash
# From ESP-IDF CMD prompt (esp32s3 is equally valid everywhere esp32c3 appears):
idf.py set-target esp32c3
idf.py build
idf.py flash monitor

# From Git Bash (via wrapper):
cmd.exe //c "build.bat set-target esp32c3"
cmd.exe //c "build.bat build"
cmd.exe //c "build.bat flash monitor"
```

Output: `build/TailFirmware.bin` — roughly 0.7-0.8 MB, but the exact size depends
on the target and the build, so don't quote a number: `idf.py size` prints it.

### Dual-target layout

`set-target` selects everything chip-specific; there is nothing else to edit.
All application code is shared — only these three places branch, and a change to
one usually needs the matching change in the others:

| | ESP32-C3 | ESP32-S3 |
|---|---|---|
| Pin map (`main/config/pin_config.h`) | SDA/SCL 7/8, TMC TX/RX 3/9, EN 5, DIAG 6, strip 4, status 10 | SDA/SCL 8/9, TMC TX/RX 17/18, EN 16, DIAG 15, strip 38, status 21 |
| `sdkconfig.defaults.<target>` | 2 MB flash, `partitions.csv` | 4 MB flash, `partitions_4mb.csv` |
| App slots | 960 KB ×2 | 1984 KB ×2 |
| Cores | 1 (RISC-V) | 2 (Xtensa) — task stacks scaled ×1.5 in `main.c` |

`pin_config.h` `#error`s on any other target rather than silently inheriting a
map. Both targets are built in CI (`firmware-build` matrix).

The cross-task hand-offs (`CommandQueue`, `MotionBus`, `FftBuffer`) are
single-producer/single-consumer with release/acquire ordering and are safe on
two cores, which the host stress test exercises on genuinely parallel threads
under ThreadSanitizer. The shared *subsystem state* is not: `motion`, `led_rend`
and `config` swap and resize `MotionPattern`/`LedEffect` instances and
`LedMatrix::coords_` with no lock, relying on single-core priority preemption,
so all three are pinned to core 0 on both targets (`main.c`). Unpinning them
needs retirement slots for those three objects first.

## Test

Host-based unit + integration tests (no hardware/ESP-IDF needed) live in `test/`.
They compile the real C++ app logic against fakes for the ESP-IDF hardware layer.

```bash
./test/run_tests.sh        # configure + build + ctest (Git Bash / Linux / macOS)
# or manually:
cmake -S test/host -B build/host-tests -G Ninja
cmake --build build/host-tests
ctest --test-dir build/host-tests --output-on-failure
```

Requires a C++20 host compiler (e.g. MSYS2 `g++`), CMake, CTest. Unit tests are in
`test/host/unit/`, feature/BLE-path integration tests in `test/host/integration/`,
ESP-IDF fakes in `test/host/fakes/`. See `test/README.md`. Companion-app protocol
improvement backlog: `docs/companion-app-improvements.md`.

## Project Structure

```
main/
  main.c                    Entry point, FreeRTOS task creation, NVS init
  app_bridge.cpp            C/C++ bridge - owns global subsystem instances, builds BLE read payloads
  stepper.c/h               TMC2209 UART driver for 4 stepper motors (VACTUAL velocity + StallGuard/DRV_STATUS read-back, shared-EN freewheel)
  ble_service.c/h           NimBLE GATT server: 14 FF00 characteristics + Battery (0x180F) + Device Info (0x180A)
  drivers/
    i2c_mux.c/h             TCA9548A I2C multiplexer
    encoder_driver.c/h      AS5600 magnetic rotary encoder (drift correction + rehome only; see motion/encoder_assist)
    imu_driver.c/h          BMI270 IMU (accel, gyro, tap detection)
    imu_tap.h               Tap-detector tuning shared with the FF01/FF06 wire contract
    led_strip_driver.c/h    WS2812B via RMT peripheral
    battery_adc.c/h         Pack voltage through a divider on ADC1 (unpopulated -> "not present", never 0 %)
  motion/
    motion_profile.cpp/h    Per-motor jerk-limited profile (max vel/accel/jerk), dead-reckoned position
    axis_controller.cpp/h   2 open-loop motor halves = 1 axis, each driven by a MotionProfile
    axis_mixer.cpp/h        Logical X/Y <-> physical axis targets (MOT-0); default identity
    encoder_assist.cpp/h    AS5600 slow correction + rehome of the dead-reckoned position (MOT-1). NOT a closed loop; off by default
    behavior_engine.cpp/h   State machine above patterns: moods + trigger table (MOT-6). MOTION_PRECEDENCE is defined here
    keyframe_sequence.cpp/h Uploaded timed pose lists, 4 flash slots (MOT-8)
    motion_bus.cpp/h        SPSC snapshot of positions/velocities/gravity/taps for LED effects
    motion_system.cpp/h     2 axes + 2 IMUs + pattern dispatch + StallGuard stall latch (go-limp)
    motion_pattern.h        Virtual base class for patterns
    pid_controller.cpp/h    PID with anti-windup — DEAD CODE. Open-loop motors never used it; protocol v6 removed the PID fields from the config, FF01 (MCMD_SET_PID retired to UNKNOWN_CMD) and FF06. Kept only for its unit test.
    patterns/               11 patterns: static, wagging, loose, idle_sway, excited_wag,
                            circle, figure_eight, shiver, audio_wag, heartbeat, keyframe
                            (+ envelope.h)
  led/
    color.cpp/h             RGB/HSV types, blend helpers
    palette.cpp/h           6 built-in palettes + 4 user slots
    noise.h                 Value noise for the ambient effects
    led_matrix.cpp/h        Ring config -> coordinate mapping, output stage (brightness/limit/gamma)
    layer_compositor.cpp/h  Layer stack with 7 blend modes + per-layer opacity
    led_effect.h            Virtual base class with flip/mirror
    render_scheduler.cpp/h  5-60 Hz frame pacing; drops late frames rather than catching up
    animation_clip.cpp/h    Stored multi-frame image (header + CRC)
    animation_storage.cpp/h LittleFS `anim` partition, 4 slots
    effects/                18 effects: rainbow, static_color, image, audio_power, audio_bar,
                            audio_freq_bars, beat_pulse, fire, breathing_glow, comet, twinkle,
                            gradient_scroll, plasma, candle_flicker, motion_glow, tap_ripple,
                            gravity_level, animation
  ble/
    ble_protocol.h          All command IDs, characteristic UUIDs, wire layouts
  config/
    config_types.h          Shared C/C++ config structs
    config_manager.cpp/h    BLE command dispatch, NVS persistence, profiles, pattern/effect factories
    command_queue.h         Lock-free SPSC queue: nimble_host produces, config task consumes
    fft_buffer.cpp/h        Double-buffered FFT data from BLE stream
    param_descriptor.h      Per-parameter name/min/max/default/unit tables (FF0D)
    pin_config.h            Per-target pin maps; `#error`s on an unknown target
  system/
    device_info.h           Firmware version + DIS strings (single source of truth)
    sys_metrics.c/h         Uptime, heap, per-task stack headroom for FF0C
    battery_monitor.cpp/h   Median filter + low/critical policy (dim LEDs, derate motion)
    ota_manager.cpp/h       OTA transfer state machine, streaming CRC, rollback bookkeeping
    ota_flash.c/h           The only file that knows `esp_ota_ops` exists
```

## Key Details

- Targets: ESP32-C3 (RISC-V single-core, 160 MHz) and ESP32-S3 (Xtensa dual-core,
  240 MHz); BLE 5.0 on both. Pin numbers below are the C3's — see the
  dual-target table above for the S3 column, and `pin_config.h` for both.
- Language: C for drivers, C++20 for application logic (patterns, effects, config)
- BLE stack: NimBLE (ESP-IDF component)
- Steppers: 4x TMC2209 on a shared UART bus (addresses 0-3 via strapped MS1/MS2),
  velocity via VACTUAL. **Open-loop**: no PID, no closed loop; motion is shaped
  entirely by a per-motor jerk-limited profile (configurable max vel/accel/jerk).
  RX is bound for StallGuard/DRV_STATUS read-back; on a detected stall the shared
  EN line is released so all motors freewheel (latched off until re-enabled).
  Pins: UART TX 3 / RX 9, shared EN 5 (active low)
- Encoders: the AS5600s are **not** feedback. `EncoderAssist` (off by default,
  `MCMD_SET_ENCODER_CFG`) nudges the profile's dead-reckoned position toward the
  measured angle at a hard-capped rate, and adopts it outright on a rehome. With
  assist on, FF02's positions are measured; with it off they are dead-reckoned
  and the encoders are not touched at all.
- LED strip: WS2812B via RMT, GPIO 4
- I2C bus: 400 kHz, GPIO 7 (SDA) / 8 (SCL), via TCA9548A mux
- Status LED: GPIO 10
- Debug: USB-Serial-JTAG (built in on both chips)

## BLE Service

Service UUID: `0000FF00-0000-1000-8000-00805F9B34FB`

| Characteristic | UUID | Properties | Purpose |
|---|---|---|---|
| Motion Command | `FF01` | Write, Write No Rsp | 21 commands (`0x01`-`0x16`, `0x04` retired): pattern select/params, motor + axis config, motion limits, calibration, motor enable, motor/gentle scale, keyframe-sequence upload+select (`0x0C`-`0x0F`), behavior engine (`0x10`-`0x13`), tap config, axis mix, encoder assist |
| Motion State | `FF02` | Read, Notify | Motor positions (encoder-measured where assist is on, else dead-reckoned), logical positions, gravity, behavior state |
| LED Command | `FF03` | Write, Write No Rsp | 15 commands (`0x01`-`0x0F`): layer config, effect params, transforms, opacity, output config, frame rate, image + animation upload |
| LED State | `FF04` | Read, Notify | Active layers and parameters |
| FFT Stream | `FF05` | Write No Rsp | Real-time audio FFT data at 30fps |
| System Config | `FF06` | Read, Write | System info + capabilities (framed `[tag][len]` blocks in v6), motion/tuning/OTA/tap/axis-mix/identity blocks; device name, bonds, descriptor select |
| System State | `FF07` | Read, Notify | Readable event ring: tap, config-changed, stall, driver-fault (`0x05`–`0x0C`), battery (`0x0D`–`0x0F`), behavior (`0x10`) events |
| Profile | `FF08` | Read, Write | Save/load/delete/rename config profiles |
| Command Result | `FF09` | Read, Notify | Per-command ACK/error `[char_lo][cmd_id][result][seq]` |
| LED Direct | `FF0A` | Write No Rsp | Direct pixel streaming `[start:u16][rgb...]`, bypasses effects |
| Motion Target | `FF0B` | Write No Rsp | Live logical motion targets (the "puppet" stream); times out to idle |
| Diagnostics | `FF0C` | Read, Notify | Health snapshot: heap, uptime, stall/overrun counts, sensor + driver health |
| Param Descriptors | `FF0D` | Read, Notify | Per-parameter name/min/max/default/unit for one selected pattern or effect (SYS-9) |
| OTA Data | `FF0E` | Write No Rsp, Read, Notify | OTA firmware image in, offset echo out |

Plus the standard **Battery Service** (`0x180F`/`0x2A19`) and **Device Information
Service** (`0x180A`).

All write commands use: `[command_id: u8] [payload...]`. Every write except the FF05
(FFT), FF0A (pixel) and FF0B/FF0E (motion-target / OTA) streams is acknowledged on
FF09. Protocol version is `BLE_PROTOCOL_VERSION` (FF06 read, first byte), currently
**6**: v6 retired the PID commands/fields and framed the FF06 trailing blocks with a
`[tag][len]` prefix. Direct LED drive: enable with FF03 `LCMD_SET_DIRECT_MODE`, then
stream frames on FF0A. See `docs/ble-protocol.md`.

## Conventions

- C11 for drivers, C++20 for application code
- ESP-IDF component model (`main/` directory)
- `extern "C"` guards on all headers shared between C and C++
- `vec3_t` and all shared types in `config/config_types.h`
- FreeRTOS tasks: nimble_host(event, pri5), motion(100Hz, pri4), LED(30Hz, pri3), config(1Hz, pri2), status LED(pri1)
- NVS namespace `tail_cfg` for active config, `tail_prof0`-`tail_prof3` for profiles
- BLE command IDs defined in `ble/ble_protocol.h`
- Effect/pattern extensibility: derive from `LedEffect`/`MotionPattern`, add factory case in config_manager

---
> Source: [sverredraaisma/TailFirmware](https://github.com/sverredraaisma/TailFirmware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
