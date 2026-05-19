## cycle-accurate-streaming-pipeline

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Run Commands

```bash
# Build and simulate (default: always-ready output)
make run

# Build and simulate with specific output stall mode
make run always    # out_ready always high
make run periodic  # out_ready high for 5 cycles, low for ~10 cycles
make run random    # out_ready randomly stalled

# Build without running
make all

# View waveforms (generates trace.vcd first)
make view-trace
surfer trace.vcd &  # open in Surfer GUI

# Clean build artifacts and trace
make clean
```

Verilator compiles `rtl/stream_pipeline.sv` into C++ (`build/`), then links with `sim/stream_pipeline_tb.cpp` to produce `build/sim`. The `trace.vcd` file is generated at the repo root on every run.

## Architecture

### RTL: `rtl/stream_pipeline.sv`

A parameterized ready-valid streaming pipeline. Key parameters:
- `DATA_WIDTH` (default 32): bit width of the data bus
- `NUM_STAGES` (default 4): number of pipeline stages

The pipeline uses `(NUM_STAGES+1)`-wide arrays (`valid[]`, `ready[]`, `data[]`) indexed `0..NUM_STAGES`. Index 0 is the input boundary; index `NUM_STAGES` is the output boundary. Each stage registers its output on `posedge clk` and only advances when a valid handshake occurs (`ready && valid`). Back-pressure propagates via combinational `ready[i] = !valid[i+1] || ready[i+1]`. Each stage increments `data` by 1 — the testbench uses this as a correctness check (output must equal input + 4 for 4 stages).

### Testbench: `sim/stream_pipeline_tb.cpp`

C++ Verilator testbench with two independently controllable modes:
- **InputMode** (`CONTINUOUS`, `BURSTY`, `RANDOM`): currently hardcoded to `CONTINUOUS` in `main()`
- **OutputMode** (`ALWAYS_READY`, `PERIODIC_STALL`, `RANDOM_STALL`): selected at runtime via CLI argument

A `pending_queue` tracks in-flight transactions. When the input handshake fires, the expected output (`in_data + NUM_STAGES`) is pushed. When the output handshake fires, the actual output is `assert`-compared against the front of the queue.

The testbench does not include a VCD viewer — after simulation, open `trace.vcd` in Surfer or GTKWave manually.

## Tools Required

- **Verilator** (`brew install verilator`) — mandatory
- **clang++ / g++** — provided by Xcode command-line tools
- **Surfer** (`brew install surfer`) — optional, for waveform viewing

---
> Source: [MarwanFathy99/Cycle_Accurate_Streaming_Pipeline](https://github.com/MarwanFathy99/Cycle_Accurate_Streaming_Pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
