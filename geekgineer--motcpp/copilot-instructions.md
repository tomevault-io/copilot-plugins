## motcpp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**motcpp** is a modern C++ multi-object tracking (MOT) library implementing 10 state-of-the-art tracking algorithms (SORT, ByteTrack, OC-SORT, DeepOC-SORT, StrongSORT, BoT-SORT, BoostTrack, HybridSORT, UCMCTrack, OracleTrack). It is a C++ port of the Python BoxMOT library, targeting 10–100x speed improvements over Python equivalents.

**License**: AGPL-3.0  
**Requires**: C++17, CMake 3.20+, OpenCV 4.x, Eigen3, yaml-cpp  
**Optional**: ONNX Runtime (ReID features), spdlog, GoogleTest (auto-fetched)

## Build Commands

```bash
# Standard build
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)

# Build with tests (default: ON)
cmake -B build -DMOTCPP_BUILD_TESTS=ON -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j$(nproc)

# Run tests
cd build && ctest --output-on-failure

# Run a single test binary
./build/tests/motcpp_tests --gtest_filter=TestSuiteName.TestCaseName

# Coverage report
cmake -B build -DMOTCPP_COVERAGE=ON
cmake --build build --target coverage
# Output: build/coverage_report/index.html

# Build shared library
cmake -B build -DBUILD_SHARED_LIBS=ON

# Disable ONNX (motion-only trackers)
cmake -B build -DMOTCPP_ENABLE_ONNX=OFF
```

**Key CMake options**:
| Option | Default | Purpose |
|--------|---------|---------|
| `MOTCPP_BUILD_TESTS` | ON | GoogleTest unit tests |
| `MOTCPP_BUILD_BENCHMARKS` | OFF | Benchmark targets |
| `MOTCPP_BUILD_EXAMPLES` | ON | Example binaries |
| `MOTCPP_BUILD_TOOLS` | ON | `motcpp_eval` CLI tool |
| `MOTCPP_ENABLE_ONNX` | ON | ONNX Runtime for ReID |
| `MOTCPP_COVERAGE` | OFF | gcov/lcov coverage |
| `BUILD_SHARED_LIBS` | OFF | Static by default |

## Architecture

### Data Flow

```
Detections (N×6: x1,y1,x2,y2,conf,cls)
         ↓
   BaseTracker::update()
         ↓
  ┌──────────────────────────────────┐
  │  1. Pre-process & validate      │
  │  2. Predict existing tracks     │
  │  3. Compute cost matrix         │
  │     (IoU / appearance / hybrid) │
  │  4. Hungarian assignment        │
  │  5. Update matched tracks       │
  │  6. Birth/death track states    │
  └──────────────────────────────────┘
         ↓
Tracks (M×8: x1,y1,x2,y2,id,conf,cls,det_idx)
```

### Key Abstractions

**`BaseTracker`** (`include/motcpp/tracker.hpp`, `src/tracker.cpp`)  
Abstract base class for all trackers. Defines `update(dets, img)` and `reset()`. Manages per-class tracking, input validation, and visualization utilities. All 10 trackers inherit from this.

**Motion Models** (`include/motcpp/motion/`, `src/motion/`)  
- `KalmanFilterXYSR` — state: [x, y, scale, aspect ratio]; used by SORT, OC-SORT, BoostTrack  
- `KalmanFilterXYAH` — state: [x, y, aspect ratio, height]; used by ByteTrack, StrongSORT, OracleTrack  
- `UCMCKalmanFilter` — ground-plane model for camera motion compensation  
- `BaseKalmanFilter` — common interface: `initiate`, `predict`, `update`, `project`

**Association** (`include/motcpp/utils/`, `src/utils/`)  
- `iou.hpp` — IoU, GIoU, DIoU, CIoU computations  
- `matching.hpp` — Hungarian algorithm (linear assignment), cascade matching  
- `association.hpp` — Mahalanobis and Gaussian gating distances  

**Appearance / ReID** (`include/motcpp/appearance/`, `src/appearance/`)  
- `ReIDBackend` — abstract interface for feature extraction  
- `ONNXBackend` — ONNX Runtime implementation  
- Used by: DeepOC-SORT, StrongSORT, BoT-SORT, BoostTrack, HybridSORT

**Configuration** (`include/motcpp/config.hpp`, `src/config.cpp`)  
YAML-based config loading. Per-tracker default configs live in `configs/trackers/*.yaml`. Supports float/int/bool/string parameters.

### Tracker Categories

| Motion-only | Appearance (ReID) required |
|-------------|---------------------------|
| SORT, ByteTrack, OC-SORT, UCMCTrack, OracleTrack | DeepOC-SORT, StrongSORT, BoT-SORT, BoostTrack, HybridSORT |

### Source Layout

```
include/motcpp/       ← Public headers (installed with library)
  motcpp.hpp          ← Single-include entrypoint
  tracker.hpp         ← BaseTracker interface
  trackers/           ← One header per tracker
  motion/             ← Kalman filter interfaces
  appearance/         ← ReID backend interfaces
  utils/              ← IoU, matching, ops
  association/        ← LAP solver

src/                  ← Implementations (.cpp)
  trackers/ motion/ appearance/ utils/ data/

tests/                ← GoogleTest unit tests (per module)
configs/trackers/     ← YAML defaults per tracker
cmake/                ← ONNX download, model export, package config
scripts/              ← Benchmarking, ReID export, eval helpers
assets/               ← MOT17 mini datasets for testing
```

## Code Style

The repo uses `.clang-format`. Run before committing:
```bash
clang-format -i src/**/*.cpp include/**/*.hpp
```

## Adding a New Tracker

1. Add header to `include/motcpp/trackers/my_tracker.hpp` inheriting `BaseTracker`
2. Add implementation to `src/trackers/my_tracker.cpp`
3. Include it in `include/motcpp/motcpp.hpp`
4. Add a YAML config to `configs/trackers/`
5. Register in `src/tracker_factory.cpp` (if a factory exists) or the CMakeLists source list
6. Add tests in `tests/test_my_tracker.cpp`

---
> Source: [Geekgineer/motcpp](https://github.com/Geekgineer/motcpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
