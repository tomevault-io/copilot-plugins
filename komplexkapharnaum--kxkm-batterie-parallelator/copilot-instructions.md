## kxkm-batterie-parallelator

> - This AGENTS.md is the workspace-wide instruction source. Do not add a second global .github/copilot-instructions.md unless intentionally replacing this file.

# Project Guidelines

## Scope
- This AGENTS.md is the workspace-wide instruction source. Do not add a second global .github/copilot-instructions.md unless intentionally replacing this file.
- Repository focus: BMU firmware for parallelized battery packs (up to 16 channels), with PlatformIO/Arduino code in firmware/ and ESP-IDF code in firmware-idf/.
- Safety is mandatory: do not weaken battery protection logic, topology checks, authentication checks, or reconnect behavior.
- Prefer focused diffs in firmware/src/ and firmware-idf/. Avoid unrelated edits in hardware folders unless explicitly requested.

## Code Style
- Keep naming/comment language consistent with surrounding files (French-first in legacy firmware paths).
- Preserve existing API shapes and file-local style; avoid unrelated refactors.
- Prefer small, reviewable patches and reuse existing helpers before introducing new abstractions.
- For diagnostics, use DebugLogger categories over ad-hoc Serial.print.

## Build and Test
- Quick validation before substantial firmware edits:
  - pio test -e sim-host
- Additional checks when scope grows:
  - pio run -e kxkm-s3-16MB
  - scripts/check_memory_budget.sh --env kxkm-s3-16MB --ram-max 75 --flash-max 85
  - cd firmware-idf && idf.py build
- PlatformIO gotcha:
  - Keep framework=arduino in [arduino_base] only; do not move it to global [env] (breaks sim-host/native tests).
- Canonical command references:
  - [CLAUDE.md](CLAUDE.md)
  - [README.md](README.md)
  - [scripts/ci/README.md](scripts/ci/README.md)

## Architecture
- Safety core:
  - firmware/src/main.cpp (orchestration + fail-safe topology behavior)
  - firmware/src/BatteryParallelator.* (protection state machine)
  - firmware/src/BatteryManager.* (aggregated state/metrics)
- I2C and peripherals:
  - firmware/src/I2CMutex.h (global I2C mutex + I2CLockGuard)
  - firmware/src/INAHandler.* (INA237 path)
  - firmware/src/TCAHandler.* (TCA9535 path)
- Web mutation surface:
  - firmware/src/WebServerHandler.*
  - firmware/src/WebRouteSecurity.*
  - firmware/src/WebMutationRateLimit.*
  - firmware/src/BatteryRouteValidation.*

## Conventions
- Keep French comments/identifiers consistent with surrounding code.
- All I2C access must go through I2CLockGuard.
- Keep I2C recovery semantics intact (kI2CRecoveryThreshold = 5 consecutive failures).
- Validate battery/sensor indexes before any array/driver access.
- Preserve NAN/early-return semantics for sensor and bus faults.
- Keep periodic FreeRTOS tasks non-blocking.
- Maintain topology constraint Nb_TCA * 4 == Nb_INA; mismatch must remain fail-safe (all batteries OFF).
- Keep thresholds centralized in firmware/src/config.h (mV/mA conventions).
- Prefer DebugLogger categories (KxLogger.h) over ad-hoc Serial.print.

## Copilot Workflow
- Start by loading the scoped instruction file for the area being edited:
  - [.github/instructions/firmware-safety.instructions.md](.github/instructions/firmware-safety.instructions.md)
  - [.github/instructions/webserver-safety.instructions.md](.github/instructions/webserver-safety.instructions.md)
  - [.github/instructions/firmware-idf-display-safety.instructions.md](.github/instructions/firmware-idf-display-safety.instructions.md)
  - [.github/instructions/tests-python.instructions.md](.github/instructions/tests-python.instructions.md)
  - [.github/instructions/ml-pipeline.instructions.md](.github/instructions/ml-pipeline.instructions.md)
  - [.github/instructions/plan-todo-implementation.instructions.md](.github/instructions/plan-todo-implementation.instructions.md)
- For firmware safety changes, inspect the relevant code path and the closest sim-host tests before proposing edits.
- Reuse existing validation/security helpers and keep diffs minimal.
- Update nearby tests when behavior changes (especially protection, routes, auth, rate limiting).
- For risk-focused review work, prefer narrow agents in .github/agents/ over broad generic reviews.
- Link to existing documentation rather than embedding long duplicated guidance in code changes.

## Link-First References
- Safety and implementation context: [CLAUDE.md](CLAUDE.md)
- Project overview and setup: [README.md](README.md)
- Kill_LIFE specs and gates: [specs/00_intake.md](specs/00_intake.md), [specs/01_spec.md](specs/01_spec.md), [specs/02_arch.md](specs/02_arch.md), [specs/03_plan.md](specs/03_plan.md), [specs/04_validation.md](specs/04_validation.md)
- Current execution plan and safety refactor tracking: [plan/refactor-safety-core-web-remote-1.md](plan/refactor-safety-core-web-remote-1.md)
- CI/QA orchestration: [scripts/ci/README.md](scripts/ci/README.md), [.github/workflows](.github/workflows)
- ML governance and artifacts: [docs/ML_BATTERY_HEALTH_SPEC.md](docs/ML_BATTERY_HEALTH_SPEC.md), [models](models)
- QA environments and remote proof flow: [docs/QA_CICD_ENVIRONNEMENTS.md](docs/QA_CICD_ENVIRONNEMENTS.md), [docs/QA_REMOTE_PROOF_LATEST.md](docs/QA_REMOTE_PROOF_LATEST.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KomplexKapharnaum) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
