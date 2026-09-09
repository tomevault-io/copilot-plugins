## motorcontrol

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Firmware for an STM32F446-based 3-phase motor controller (FOC / field-oriented control) driving a DRV8323 gate driver, written for the hardware at https://github.com/bgkatz/3phase_integrated. Bare-metal C with the ST HAL, no RTOS.

## Build

```
make            # outputs build/motorcontrol.elf/.bin/.hex (arm-none-eabi-gcc + make, both on PATH)
make DEBUG=1    # -Og/-g3 debug build
make flash      # flash over ST-Link (STM32_Programmer_CLI or st-flash)
make clean
```

The STM32CubeIDE project also still works. There are no automated tests; verification is a clean build plus bench testing by the maintainer. For pure math changes (e.g. `math_ops.c`), a bit-faithful float32 Python simulation (numpy `float32`, C-style truncating casts) compared against float64 references has been used to verify before flashing.

**CubeMX caveat:** `adc.c` and `tim.c` contain hand edits *outside* the `USER CODE` blocks (triple injected simultaneous ADC mode, TIM1 TRGO on update). Regenerating from `motorcontrol.ioc` will silently revert them. Don't regenerate.

## Real-time architecture (the most important thing to understand)

Everything is organized around a hard 40 kHz (25 µs, `DT`) control interrupt and a slow main loop:

- **`TIM1_UP_TIM10_IRQHandler`** (`stm32f4xx_it.c`, priority 0) runs every PWM cycle: kick IWDG → `analog_sample` → `ps_sample` → `run_fsm` → `can_tx_rx`. TIM1's update event also hardware-triggers the ADCs (triple injected simultaneous mode), so `analog_sample` just polls JEOC and reads fresh, PWM-synchronized samples.
- **Main loop** (`main.c` + `run_fsm_slow` in `fsm.c`, ~1 ms cadence) does everything slow: all `printf`, all flash writes, DRV8323 fault polling. The ISR hands work over via the `volatile` `*_pending` flags on `FSMStruct`.

Invariants to preserve:

- **Never `printf`, write flash, or do blocking SPI transactions in the ISR path** (`run_fsm` and below). The only exceptions are the encoder SPI read in `ps_sample`, the DRV gate enable/disable on state transitions, and short calibration milestone prints that occur at zero-current moments.
- **Flash writes stall the whole CPU for up to ~2 s** (128 KB sector erase, single-bank part). They must go through `save_preferences()` (`fsm.c`), which kicks the watchdog first, and only run in states where the gate drive is off.
- **The IWDG (~4 s timeout) is kicked by the control ISR.** Don't add anything that stalls longer than the flash erase. Its timeout cannot be shortened below ~2.7 s worst-case because of the erase stall and LSI tolerance (17–47 kHz).
- **SPI1 (DRV8323) is shared** between the ISR (gate enable/disable on state transitions) and the main loop (fault register reads). Main-loop access must mask `PWM_ISR` around the SPI transaction, print afterwards.
- **Fault handlers call `fault_shutdown()`** — gate drive off by GPIO (never SPI; the bus may be wedged) plus TIM1 MOE clear. Keep it register-writes-only.

## Structure

- Global singleton structs (`controller`, `comm_encoder`, `state`, `comm_encoder_cal`, `can_tx/rx`, `drv`, `prefs`) are defined in `main.c` and extern'd everywhere via `structs.h`.
- `hw_config.h` — pin/peripheral/timer mapping and hardware constants. Most porting happens here.
- `user_config.h` — user parameters as macros aliasing `__float_reg[]`/`__int_reg[]`, persisted by `preference_writer` as CRC-checked, sequence-numbered blocks ping-ponged between flash sectors 6 and 7 (a save never destroys the previous config; legacy pre-CRC configs are imported on first boot). Boot-time sanitization in `main.c` supplies defaults on an erased chip. `ENCODER_LUT` is `__int_reg[6..133]`.
- FSM (`fsm.c`): modes MENU/CALIBRATION/MOTOR/SETUP/ENCODER/FAULT. Transitions execute in the ISR (`run_fsm`); `fsm_enter_state`/`fsm_exit_state` enable/disable the gate drive. Serial input arrives via `HAL_UART_RxCpltCallback` → `update_fsm`.
- Latched faults: `check_faults` (`foc.c`, called every ISR cycle) latches overvoltage (`V_BUS_FAULT`) and gross overcurrent bits into `controller.fault`; any nonzero fault forces FAULT_MODE (gates off) until the user explicitly exits to the menu, which is what clears it. Faults are RAM-only (cleared by power cycle). `V_BUS_MAX` remains a soft torque cut below the hard fault threshold.
- FOC (`foc.c`): `commutate` is the current loop (dq0 → PI with anti-windup → SVM → `set_dtc`); `torque_control` is the outer position/velocity/torque law; `field_weaken` runs between them. `PHASE_ORDER` (set by calibration) swaps both the PWM channel and current-sense assignment.
- Trig (`math_ops.c` + `lookup.h`): `sincos_lut` does multiply-based range reduction + linear interpolation. `sin_tab` has 513 entries on the 2π/512 grid — the indexing and the table grid must match; the table header comment documents this.
- Position sensing (`position_sensor.c`): velocity is computed from **integer count** differences (`count_buff`), not the float multi-turn angle, to keep resolution independent of position magnitude. Multi-turn counts are exact to ±32768 turns.

## CAN protocol

The spec-of-record is the comment table in `can.h`. Summary: 11-bit ID = `(opcode << 7) | node_id` (node ID 0–127). Opcode 0 is the legacy packed 8-byte protocol (replies to `CAN_MASTER`), so plain-ID masters work unchanged. Float opcodes: SET_PV / SET_GAINS / SET_TFF (clamped, NaN-rejected, reset the command timeout), SET_REPLY_FORMAT (RAM only), READ_VARS and telemetry replies on `(0xF<<7)|CAN_ID`. Reads deliberately do **not** reset the CAN command timeout. Readable variables are the `CAN_VAR_*` IDs; add new ones in `can_read_variable()`. IDs 0–13 are live measurements; 14–23 are read-only config and status (setup values, `VERSION_NUM`, and the FSM state), which a host has no other way to see — the setup parameters otherwise exist only on the serial setup screen. Adding IDs is safe for older masters: unknown IDs read back NaN, so a host can feature-detect by reading one and testing for NaN.

## Conventions

- Positions/velocities on the CAN bus and in `torque_control` are output-shaft quantities (divided by gear ratio `GR`); `controller` d/q quantities are motor-side.
- All wire-format floats are float32 little-endian.
- New float inputs from outside (CAN, serial) must be clamped and NaN-checked before touching the control loop.
- Indentation is tabs; code style follows the existing files (compact, comment only where hardware behavior isn't obvious).

---
> Source: [bgkatz/motorcontrol](https://github.com/bgkatz/motorcontrol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
