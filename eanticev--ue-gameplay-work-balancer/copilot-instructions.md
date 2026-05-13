## ue-gameplay-work-balancer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Gameplay Work Balancer (GWB) is an Unreal Engine plugin that helps distribute and balance intensive computational work across multiple frames to maintain performance targets. It's designed to prevent gameplay hitches by managing the execution time of background tasks, AI processing, and other computationally intensive systems.

## Development Commands

### Building and Testing
- **Build Plugin**: This is an Unreal Engine plugin - build through the Unreal Editor or using UBT (Unreal Build Tool)
- **Run Tests**: Execute automation tests via UE Editor's Session Frontend or command line:
  ```
  UnrealEditor.exe [ProjectPath] -ExecCmds="Automation RunTests GWB" -unattended -nopause -testexit="Automation Test Queue Empty"
  ```
- **Run Specific Test**: Target individual test classes like `GWBManagerTests`, `GWBSchedulerTests`, `GWBExtensionsTests`

### Module Structure and Dependencies

The plugin consists of 4 modules in dependency order:

1. **GWBTimeSlicer** (Foundation) - Low-level time slicing utilities and `BUDGETED_FOR_LOOP` macro
2. **GWBRuntime** (Core) - Main work balancing system, depends on GWBTimeSlicer
3. **GWBCustomNodesRuntime** - Blueprint node runtime support, depends on GWBRuntime  
4. **GWBEditor** (Editor-only) - Blueprint graph integration, depends on both runtime modules

All modules use C++20 standard and include comprehensive automation test integration.

## Architecture and Key Systems

### Core Components

- **UGWBManager** - Central singleton for scheduling/executing work units and managing work groups
- **UGWBSubsystem** - Engine subsystem managing GWB system lifecycle and providing component access
- **UGWBScheduler** - Controls work cycle timing and delegates execution scheduling
- **UGWBTimeSlicer** - Low-level time budget tracking and enforcement for individual operations
- **FGWBWorkUnit** - Individual work items with callback delegates and state tracking
- **UGWBWorkUnitHandle** - Lightweight handle providing user interface to scheduled work

### Extension System

The plugin features a sophisticated **modifier system** for runtime behavior customization:

- **ValueModifier\<T\>** - Type-erased function-based modifier system (C++ only, no Blueprint support)
- **FModifierManager** - Manages budget and priority modifiers using type erasure instead of virtual inheritance
- **Extensions API** - Allows registration of custom modifiers that can alter frame budgets, priorities, and respond to work lifecycle events

### Time Slicing Utilities

- **FGWBTimeSlicedScope** / **FGWBTimeSlicedLoopScope** - Manual time slicing for custom loops
- **BUDGETED_FOR_LOOP** macro - Convenient wrapper for time-sliced iteration over containers
- **FBudgetedLoopHandle** - Provides early loop termination capabilities

### Custom K2 Node with Value Capture System

**UK2Node_GWBScheduleWork** is a custom K2 node that uses the **GWBCustomNodesRuntime** module which includes a sophisticated caching system for Blueprint latent node context:
- Captures and preserves variable context across frame boundaries
- Enables Blueprint nodes to maintain state during asynchronous work execution

## Development Guidelines

### Testing Framework

- Use **FScopedCVarOverride** template for temporarily modifying console variables during tests
- Extend real implementations with `TEST_` prefixed methods for mocking
- Tests use Unreal's BDD-style spec framework (`BEGIN_DEFINE_SPEC`)
- Test utilities located in dedicated `Tests/` folders within each module

### Console Variables (CVars)

Runtime behavior controlled via CVars defined in `Source/GWBRuntime/Public/CVars.h`:
- `gwb.enabled` - Global system toggle (default: true)
- `gwb.budget.frame` - Per-frame time budget in seconds (default: 0.005)
- `gwb.schedule.interval` - Work cycle interval (default: 0.0 = every frame)
- Escalation system CVars for adaptive budget scaling
- `gwb.immediateduringwork` - Immediate execution mode toggle

### Work Group Configuration

Define work groups in project INI files under `[/Script/GWBRuntime.GWBManager]`:
```ini
+WorkGroupDefinitions=(Id="CriticalSystems",Priority=100,MaxFrameBudget=0.002,MaxWorkUnitsPerFrame=5)
```

### Performance Stats

Performance monitoring available via stats system defined in `Source/GWBRuntime/Public/Stats.h`

---
> Source: [eanticev/ue-gameplay-work-balancer](https://github.com/eanticev/ue-gameplay-work-balancer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
