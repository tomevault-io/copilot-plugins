## trobot-m0

> This file explains how to work on this MSPM0G3507 firmware project safely.

# AGENTS.md

This file explains how to work on this MSPM0G3507 firmware project safely.
The project uses CMake + GCC + OpenOCD, not a CCS project layout, but the
SysConfig and DriverLib rules still matter.

## Project Shape

- `core/` contains the MSPM0 SDK surface used by this project: startup code,
  linker script, SysConfig output, DriverLib, CMSIS, FreeRTOS, and syscall glue.
- `core/trobot.syscfg` is the source of truth for clocks, pins, peripherals,
  DMA channels, interrupts, and generated initialization names.
- `bsp/` contains board support code for UART, SPI, LCD, flash, time, GPIO, and
  low-level helpers.
- `components/` contains optional reusable modules. `components/utils` is a git
  submodule and is required by the current application. It supplies the C++
  logger, task/queue wrappers, terminal, message, CRC, and VOFA helpers.
- `app/` contains the firmware entry point and application tasks.
- `bsp/include/bsp/` is the public BSP include surface. `bsp/internal/` is
  private to the BSP target and must not be included from `app/` or components.
- `*.cfg` files at the repository root select the OpenOCD probe interface.

## Runtime Architecture

The current boot and initialization sequence is:

1. The GCC startup code initializes `.data`, `.bss`, C/C++ constructors, and
   then calls `main()`.
2. `main()` calls `SYSCFG_DL_init()`, creates the `app_entrance` task, and
   starts the FreeRTOS scheduler.
3. `app_entrance()` calls `bsp_hw_init()`, initializes UART0 and its TX DMA
   queue, enables the board GPIO interrupt, initializes the logger, and creates
   application tasks.
4. `bsp_hw_init()` initializes the ST7735 LCD and verifies the W25Q128 device
   ID. A failed BSP assertion records its expression/message/file/line, breaks
   only when a debugger is attached, and then stops forever.

Keep this order in mind when adding code. A task that logs asynchronously needs
its UART initialized first. LCD and flash users need the board/SPI GPIO state
initialized first. Do not move scheduler-dependent initialization into global
C++ constructors.

The current interrupt ownership is split across layers:

- `GROUP1_IRQHandler` is owned by `app/main/main.cc` for the board key GPIO.
- `UART0_IRQHandler` through `UART3_IRQHandler` are owned by `bsp/src/uart.c`.
- `TIMG8_IRQHandler` is owned by `bsp/src/uart.c`; the generated
  `UART_RX_IDLE` one-shot timer provides the UART RX idle timeout.
- `SVC_Handler`, `PendSV_Handler`, and `SysTick_Handler` are owned by the
  FreeRTOS port.
- All other startup-vector handlers are weak defaults until a module provides
  the exact symbol.

The generated configuration currently uses an 80 MHz CPU clock, a 1 kHz
FreeRTOS tick, UART0 as `UART_DEBUG_INST`, and SPI1 for the LCD and W25Q128.
These are orientation notes only; after any SysConfig change, the generated
header and `FreeRTOSConfig.h` are the authoritative values.

## Build And Flash

Use an ARM embedded GCC toolchain.

```powershell
cmake -S . -B cmake-build-debug -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build cmake-build-debug
```

The normal output files are:

- `cmake-build-debug/trobot.elf`
- `cmake-build-debug/trobot.hex`
- `cmake-build-debug/trobot.bin`
- `cmake-build-debug/trobot.map`

The app and BSP source lists use recursive CMake globs without
`CONFIGURE_DEPENDS`. After adding, removing, or renaming a source file, rerun
the CMake configure command before building. Adding or removing a component
directory also requires reconfiguration.

This project needs an OpenOCD build with TI MSPM0 support. Before flashing,
check which OpenOCD executable is active:

```powershell
where openocd
openocd --version
```

Use the first `where openocd` result to understand which executable will run.
Then verify that it can resolve this project's MSPM0 target scripts. A quick
script-resolution check that should not initialize the adapter is:

```powershell
openocd -f daplink.cfg -c "shutdown"
```

With a DAPLink/CMSIS-DAP probe connected:

```powershell
cmake --build cmake-build-debug --target flash_and_verify
```

For other probes, use the matching root config manually:

```powershell
openocd -f xds110.cfg -c "init" -c "reset halt" -c "program cmake-build-debug/trobot.elf verify reset exit"
openocd -f stlink.cfg -c "init" -c "reset halt" -c "program cmake-build-debug/trobot.elf verify reset exit"
```

If OpenOCD reports that it cannot find `target/ti_mspm0.cfg`, the wrong OpenOCD
is being used or its script search path is wrong. If it reports that it cannot
find a matching CMSIS-DAP device, the MSPM0 scripts were found and the remaining
problem is probe/USB/driver/hardware related.

## SysConfig Rules

Treat `core/trobot.syscfg` as the peripheral configuration source, but do not
modify it by default. Only edit `core/trobot.syscfg` when the user explicitly
asks for a change that requires SysConfig, such as pins, clocks, DMA,
UART/SPI/I2C/ADC/timer setup, or interrupt ownership.

When a user-requested change does require SysConfig:

1. Edit `core/trobot.syscfg` with TI SysConfig when possible.
2. Preserve the metadata comments at the top of the file, including `@cliArgs`,
   `@v2CliArgs`, `@versions`, device, package, and SDK product.
3. Regenerate the SysConfig outputs into `core/`.
4. Re-read `core/ti_msp_dl_config.h` before using generated names in code.
5. Check that the `.syscfg`, `ti_msp_dl_config.c`,
   `ti_msp_dl_config.h`, and linker-script diffs form one consistent change
   set. A changed `.syscfg` with unchanged generated output is not complete.
6. Build the project.

Do not guess generated names. Use the local macros and function spellings from
`core/ti_msp_dl_config.h`, such as `SYSCFG_DL_init()`, `UART_DEBUG_INST`,
`DMA_UART0_TX_CHAN_ID`, `SPI1_INST`, and `GPIO_BOARD_LED_PIN`.

Be careful with these generated or SysConfig-owned files:

- `core/ti_msp_dl_config.c`
- `core/ti_msp_dl_config.h`
- `core/device_linker.lds`

It is acceptable for this repository to track those files because the CMake
build consumes them directly. However, avoid manual edits unless the change is
intentional, reviewed, and cannot reasonably be represented in SysConfig.

## Coding Rules

- Keep opening braces on the same line as functions, control-flow statements,
  types, and other block declarations.
- Use `snake_case` for functions, types, variables, parameters, and structure
  or class members. Preprocessor macro names and enum values are exempt and
  may use `UPPER_SNAKE_CASE`.
- Keep implementations as simple and direct as practical. Avoid unnecessary
  abstraction, indirection, helper layers, and duplicated state.
- Prefer short, clear variable names. Do not construct excessively long or
  complicated identifier names when a concise name communicates the scope.
- There is no strict single-line length limit. Prefer readability and do not
  wrap otherwise clear code solely to satisfy a conventional column limit.
- Prefer existing BSP APIs over directly touching DriverLib from `app/`.
- Prefer DriverLib and SysConfig-generated macros over raw register writes.
- Keep app logic in `app/`, board abstractions in `bsp/`, reusable C++ helpers in
  `components/`, and chip/toolchain integration in `core/`.
- Do not edit vendored SDK, CMSIS, FreeRTOS, or DriverLib files unless fixing a
  project-blocking integration issue.
- Preserve interrupt handler names exactly as defined by the startup file and
  `ti_msp_dl_config.h`.
- For new IRQ users, confirm NVIC enable, interrupt priority, peripheral
  interrupt enable, and the exact handler symbol.
- Be conservative with RAM. This target has 32 KiB SRAM, and the current build
  already uses a meaningful portion of it.
- Avoid large stack buffers in tasks and ISRs.
- Check formatted output lengths before passing `snprintf`/`vsnprintf` results
  to UART or DMA send functions.
- Do not call blocking or task-only FreeRTOS APIs from interrupts. Use `FromISR`
  APIs where needed.
- C++ is built without exceptions, RTTI, or `__cxa_atexit`. Do not use
  exceptions, `dynamic_cast`, or code that depends on runtime type information
  or global-object destruction.

## FreeRTOS And Concurrency

- FreeRTOS uses `heap_4.c` with a 10 KiB configured heap. Both static and
  dynamic allocation APIs are enabled, but the current app and utility task
  wrappers use dynamic allocation.
- FreeRTOS stack-depth arguments are words, not bytes. The current tick rate is
  1 kHz, preemption is enabled, and time slicing is disabled.
- Despite its name, `os::task::static_create()` currently allocates its callable
  thunk with `pvPortMalloc()` and creates the task with `xTaskCreate()`. Do not
  count it as a fully static task when budgeting RAM.
- `bsp_time_delay()` uses `vTaskDelay()` only from task context while the
  scheduler is running; before the scheduler or in an ISR it busy-waits.
  `bsp_time_get_ms()` is FreeRTOS tick time, not a persistent wall clock.
- UART RX callbacks run in the TIMG8 idle ISR. Keep them short and use queues,
  notifications, or other `FromISR` handoff APIs for substantial work.
- Each UART has one RX callback set by `bsp_uart_set_callback()`. It runs after
  the RX line becomes idle and receives at most the first 128 bytes; excess
  bytes from the same continuous receive are ignored. RX shares the generated
  16-bit TIMG8 `UART_RX_IDLE` one-shot timer across UART instances. It is armed
  only while receiving, so TIMG0 remains available to other code.
- UART asynchronous TX uses fixed packet slots protected against concurrent
  task/ISR producers. Its APIs return `false` if DMA is unavailable, a packet
  is too large, or all slots are occupied; callers that cannot tolerate loss
  must retry or provide backpressure.
- The LCD and W25Q128 share SPI1. Device helpers hold a recursive FreeRTOS
  mutex while the scheduler is active. For a multi-call LCD pixel stream, hold
  `bsp_spi_lock(SPI1_INST)` across address setup, begin/write/end, and never
  access the shared bus from an ISR.

## CMake Conventions

- Top-level `CMakeLists.txt` owns the toolchain, target executable, link flags,
  post-build artifact generation, and flash target.
- `core/CMakeLists.txt` defines `ti_core` and imports DriverLib/CMSIS DSP
  archives.
- `bsp/CMakeLists.txt` builds the board support static library.
- `components/CMakeLists.txt` auto-loads component directories with their own
  `CMakeLists.txt` and links aliases named `components::<name>`.
- `app/CMakeLists.txt` recursively includes app subdirectories and builds app
  sources as an object library.
- The top-level target uses C17/C++17 defaults, while the current `app`, `bsp`,
  and `utils` targets explicitly request C++23. New code must still obey the
  globally disabled exception/RTTI settings.

If adding a new component under `components/<name>`, provide a local
`CMakeLists.txt` and define a `components::<name>` alias so the aggregator links
it automatically.

## Submodules

Clone with submodules:

```powershell
git clone --recursive <repo-url>
```

If `components/utils` is missing:

```powershell
git submodule update --init --recursive
```

The root `.gitignore` intentionally ignores most `components/*` content while
keeping `components/CMakeLists.txt`, so remember that component code may live in
submodules.

## Validation Checklist

Before handing off a firmware change:

```powershell
cmake -S . -B cmake-build-debug -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build cmake-build-debug
```

If the task affects flashing or debug configuration:

```powershell
openocd -f daplink.cfg -c "shutdown"
```

If hardware is not connected, report validation as build/config-only. Do not
claim flashing or board behavior was verified without a connected board and
probe.

---
> Source: [lym12321/trobot_m0](https://github.com/lym12321/trobot_m0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
