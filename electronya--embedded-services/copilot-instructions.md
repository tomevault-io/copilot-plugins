## embedded-services

> Electronya reusable Zephyr RTOS service library. Consumed as a west module by

# Embedded Services — CLAUDE.md

Electronya reusable Zephyr RTOS service library. Consumed as a west module by
firmware applications (e.g. `srv-dev-app`).

## Project Intent

Provide a set of self-contained, independently testable embedded services that
any Electronya Zephyr application can include via Kconfig. Each service owns its
own thread, memory pool, and shell commands, and communicates with other services
exclusively through message queues using `SrvMsgPayload_t` (see
`src/serviceCommon/serviceCommon.h`).

The `serviceManager` is the lifecycle supervisor: it starts, stops, suspends,
and resumes all other services, and monitors them via watchdog-backed heartbeats.

## Repository Layout

```
src/
  serviceCommon/    # Shared types (Data_t, SrvMsgPayload_t, ServiceCtrlMsg_t)
  serviceManager/   # Lifecycle supervisor (start/stop/suspend/resume + watchdog)
  adcAcquisition/   # Timer-triggered ADC with DMA, RC filtering, pub-sub
  datastore/        # Thread-safe datapoint store with NVM persistence + pub-sub
tests/
  <service>/
    service/        # Service-level integration tests
    cmd/            # Shell command tests
    util/           # Utility unit tests
    filter/         # (adcAcquisition only) filter unit tests
docs/
  services/         # Per-service design docs (Sphinx + Doxygen)
```

## Services

### serviceCommon
Shared inter-service types only — no source files, header only.

### serviceManager
- Supervises all services; must be initialized last, after all services are
  registered.
- Services register themselves via the serviceManager API at init time.
- Heartbeat monitoring uses Zephyr's watchdog driver.

### adcAcquisition
- Timer-triggered, DMA-based zero-copy ADC sampling.
- 3rd-order cascaded RC digital filter (`adcAcquisitionFilter`).
- Publishes samples to subscribers via CMSIS memory pool + message queue.
- START/RESUME handled via `k_thread_start()`/`k_thread_resume()` (thread cannot
  read its control queue when not running); STOP/SUSPEND sent via control queue.

### datastore
- Six datapoint types: Binary, Button, Float, Integer, Multi-State, UnsignedInt.
- Datapoints declared at compile time with X-macros (app provides
  `datastoreMeta.h`).
- Thread-safe via message queue; optional NVM persistence.
- Shell commands for runtime read/write/inspect.

## Coding Conventions

- camelCase for functions, variables, and files.
- `UPPER_SNAKE_CASE` for macros and Kconfig symbols.
- Each service has its own `Kconfig` symbol: `CONFIG_ENYA_<SERVICE_NAME>`.
- Shell support gated behind `CONFIG_ENYA_<SERVICE_NAME>_SHELL`.
- All public headers use Doxygen `@defgroup` / `@{` / `@}` grouping.
- Copyright header on every file: `Copyright (C) <year> by Electronya`.

## Testing

Tests run via Zephyr's `twister` on the native simulator (no hardware needed).

```bash
# All tests with coverage report
./run-tests-coverage.sh

# Single service
./run-tests-coverage.sh tests/datastore
```

Coverage report is filtered to `src/` only, with LOG macro branches excluded
(they always show 50% due to compile-time conditionals).

CI runs on Gitea via `.gitea/workflows/test-and-coverage.yml`.

## West Module

Declared in `zephyr/module.yml`. Applications pull it in via their `west.yml`:

```yaml
- name: embedded-services
  url: <repo-url>
  path: apps/embedded-services
  revision: main
```

Enable in application `prj.conf`:
```
CONFIG_ENYA_EMBEDDED_SERVICES=y
```

---
> Source: [Electronya/embedded-services](https://github.com/Electronya/embedded-services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
