## generic-gantry

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Viam module implementing single-axis and multi-axis gantry components (`rdk:component:gantry`). Built on the Viam RDK framework. Two models are registered:
- `viam:generic-gantry:single-axis` — motor-driven linear axis with optional limit switches and encoder homing
- `viam:generic-gantry:multi-axis` — composes multiple single-axis gantries with sequential or simultaneous movement

## Design Priorities

1. **Correctness** — Prefer clear, provably correct code over clever optimizations. Guard invariants explicitly.
2. **Test coverage** — Every behavior and edge case should have a corresponding test. Use the `testrig` simulation framework to test realistic hardware interactions without real hardware.
3. **Reusability** — Extract shared logic into reusable components (e.g., `testrig` presets). Favor composition and interfaces over duplication.
4. **Maintainability & extensibility** — Keep packages loosely coupled with clear boundaries. New axis types or homing strategies should be addable without modifying existing code. Lean on dependency injection via Viam's `resource.Dependencies`.

## Commands

```bash
make build          # Build binary to bin/generic-gantry
make module         # Build + package as bin/module.tar.gz (binary + meta.json)
make test           # Run all tests with race detector
make lint           # go vet + golangci-lint with auto-fix
make tool-install   # Install golangci-lint and gotestsum

# Run a single test
go test -race -run TestName ./singleaxis/
```

## Architecture

### Package Structure

- **`singleaxis/`** — Core single-axis gantry. Manages a motor + optional board with limit switch pins. Converts between motor revolutions and gantry mm using `mm_per_rev`. Has a background `checkHit()` goroutine that polls limit switches at 1ms intervals. Three homing modes: encoder-only, single limit switch, dual limit switches.
- **`multiaxis/`** — Thin composition layer that delegates to an ordered list of single-axis sub-gantries. Supports `move_simultaneously` config flag.
- **`testrig/`** — Simulation framework for testing: `SimulatedMotor` (time-based position tracking with GoTo blocking), `SimulatedBoard` with `LimitSwitchPin` (derives switch state from motor position via threshold). `presets.go` has factory functions for common test rigs (printer, CNC, linear actuator).

### Key Patterns

- **Viam resource pattern**: Each package registers its model via `resource.RegisterComponent()` in `init()`. Constructor receives `resource.Dependencies` (motors, boards) and `resource.Config`. Config structs implement `Validate()` returning dependency names.
- **Concurrency**: Mutex-protected state, `operation.SingleOperationManager` for move operations, `sync.WaitGroup` for background goroutines, context-based cancellation.
- **Position math**: `gantry_mm = length * (motor_pos - min_limit) / (max_limit - min_limit)`. Speed: `motor_rpm = (mm_per_sec / mm_per_rev) * 60`.

### Testing

Three tiers: unit tests with mocked dependencies, `testrig` simulation tests, and integration tests using `testrig` presets. Tests use `go.viam.com/test` for assertions. Integration tests exercise full homing sequences and cross-limit movement.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/viam-modules) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
