## 2024hvibe

> This project is the MSPM0G3507 firmware workspace for practicing the 2024 NUEDC H control-car task.

# Agent Notes For 2024hVibe

This project is the MSPM0G3507 firmware workspace for practicing the 2024 NUEDC H control-car task.

## Current Project Facts

- CCS Theia project path: `C:\Users\3545\workspace_ccstheia\2024hVibe`
- Project template: TI `empty_LP_MSPM0G3507_nortos_ticlang`
- MCU: MSPM0G3507, package `LQFP-64(PM)`
- SDK: `C:\ti\mspm0_sdk_2_10_00_04`
- SysConfig: `C:\ti\sysconfig_1.26.2`
- Compiler from generated build: `C:\ti\ccs2020\ccs\tools\compiler\ti-cgt-armllvm_4.0.3.LTS`
- Active SysConfig source: `empty.syscfg`
- Active app entrypoint: `empty.c`
- Firmware layering: `BSP\` owns peripheral drivers and ISRs; `APP\` owns control algorithms, route state machines, commands, and UI behavior.
- Generated output currently exists under `Debug\`; treat it as inspection-only.
- `targetConfigs\MSPM0G3507.ccxml` is configured for SEGGER J-Link.
- Current detected physical probe during setup was Horco CMSIS-DAP / DAPLink on COM3, recommended for OpenOCD.

## MSPM0 Skill Rules

- Treat `empty.syscfg` as the source of truth for clocks, GPIO, PWM, UART, timers, ADC, I2C, SPI, DMA, interrupts, and pinmux.
- Do not hand-edit generated files such as `Debug\ti_msp_dl_config.c`, `Debug\ti_msp_dl_config.h`, `Debug\device_linker.cmd`, `Debug\*.mk`, `*.out`, `*.map`, `*.o`, or `*.d`.
- Do not guess generated macro names. After SysConfig generation, read `Debug\ti_msp_dl_config.h` before using new names in C code.
- Preserve SysConfig metadata comments including `@cliArgs` and `@v2CliArgs`.
- For Tianmengxing MSPM0G3507, avoid `PA21`, `PA23`, `PA02`, `PA18`, `PA10`, and `PA11` for ordinary new assignments unless the user explicitly chooses them or board documentation requires them.
- Do not change device, package, SDK, compiler, CCS version, or debug probe configuration without user confirmation.
- Keep `empty.c` as the small main entrypoint. Put hardware-facing code and peripheral ISRs in `BSP\`; put state machines and control behavior in `APP\`.
- If hardware behavior is not physically tested, report the validation level as source/static, SysConfig, build, flash, serial, or real-board behavior.

## Build And Validation Commands

Run these from PowerShell unless stated otherwise.

```powershell
python "C:\Users\3545\.agents\skills\mspm0-ccs\scripts\check_syscfg.py" "C:\Users\3545\workspace_ccstheia\2024hVibe"
& "C:\Users\3545\workspace_ccstheia\2024hVibe\tools\build.ps1"
& "C:\Users\3545\workspace_ccstheia\2024hVibe\tools\flash.ps1"
```

The project-local build script regenerates SysConfig and compiles all root, `APP\`, and `BSP\` sources with the same TI Arm Clang settings as CCS. Use it instead of the stale generated `Debug\*.mk` files after adding source directories.

Before flashing or debugging:

```powershell
python "C:\Users\3545\.agents\skills\mspm0-ccs\scripts\detect_probe.py"
python "C:\Users\3545\.agents\skills\mspm0-ccs\scripts\check_syscfg.py" "C:\Users\3545\workspace_ccstheia\2024hVibe" --probe
```

If using the currently detected CMSIS-DAP / DAPLink probe, prefer the project flash script rather than the J-Link `.ccxml`. It explicitly reset-halts before and after writing, then disables the OpenOCD work area for a direct byte comparison because the target-side Cortex-M CRC helper is unreliable over this wireless DAPLink:

```powershell
python "C:\Users\3545\.agents\skills\mspm0-ccs\scripts\openocd_debug.py" --speeds 500 "C:\Users\3545\workspace_ccstheia\2024hVibe" probe
& "C:\Users\3545\workspace_ccstheia\2024hVibe\tools\flash.ps1"
```

If the user switches to a real J-Link and confirms the `.ccxml`, CCS/DSLite flashing may use:

```powershell
dslite -c "C:\Users\3545\workspace_ccstheia\2024hVibe\targetConfigs\MSPM0G3507.ccxml" -e -r 2 -u "C:\Users\3545\workspace_ccstheia\2024hVibe\Debug\2024hVibe.out"
```

## Firmware Strategy

- First milestone: board smoke test, stable build, flash, reset, and serial debug.
- Second milestone: TB6612 motor PWM/direction driver with safe limits and emergency stop.
- Third milestone: encoder speed sampling and straight-line speed loop.
- Fourth milestone: grayscale line sensing and line-position error.
- Fifth milestone: path state machine for A->B, A->B->C->D->A, A->C->B->D->A, then four laps.
- ICM45686 IMU and display are useful, but they are not first-pass blockers.

## Hardware Safety

- Keep wheels lifted for first motor tests.
- Verify battery polarity, common ground, TB6612 VM/VCC levels, standby pin state, and motor driver thermal behavior before motion tests.
- Add a firmware stop path before high-speed tests.
- Debugger halt can freeze motor control. Prefer serial logs for live motion tests.
- Do not blame firmware until power, ground, wiring, pull-ups, signal crossover, and module mode are checked.

## Documentation

- Put datasheets and screenshots in `module_manuals\`.
- Keep pin decisions in `docs\hardware_interface.md`.
- Keep test observations in `docs\logs\bringup_log.md`.
- Keep serial commands in `docs\serial_protocol.md`.

---
> Source: [mc3545dada/2024hVibe](https://github.com/mc3545dada/2024hVibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
