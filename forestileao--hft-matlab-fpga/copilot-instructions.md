## hft-matlab-fpga

> This repository is a research prototype for a low-cost FPGA-assisted HFT pipeline. The current codebase centers on a shared-memory streaming bridge between ARM software and FPGA logic, plus a small FAST market-data feed generator/receiver pair used as a software-side harness.

# AGENTS.md

## Purpose

This repository is a research prototype for a low-cost FPGA-assisted HFT pipeline. The current codebase centers on a shared-memory streaming bridge between ARM software and FPGA logic, plus a small FAST market-data feed generator/receiver pair used as a software-side harness.

If you are making changes here, optimize for preserving the contract between:

- `cpp/src/fpga_shared_stream.h`
- `cpp/src/fast_receiver.cpp`
- `vhdl/arm_fpga_shared_stream_bridge.vhd`
- `docs/arm-fpga-shared-memory-stream.md`

Those four files define the most important integration surface in the repo.

## Repository Map

- `README.md`: project context, goals, and top-level test commands.
- `Makefile`: primary entrypoint for builds and tests. Uses Docker for all supported workflows.
- `Dockerfile`: Ubuntu 24.04 image with CMake, Boost, Git, and GHDL.
- `cpp/`: C++ utilities and tests.
- `cpp/src/templates/SimpleMD.xml`: mFAST template used to generate message types.
- `cpp/src/fast_data_feed.cpp`: TCP server that emits encoded FAST messages.
- `cpp/src/fast_receiver.cpp`: TCP client that decodes FAST messages and optionally forwards them into the FPGA MMIO bridge.
- `cpp/src/fpga_shared_stream.h`: C++ wrapper for the ARM/FPGA shared-memory ring interface.
- `cpp/tests/fpga_shared_stream_test.cpp`: host-side unit test for the MMIO wrapper using a temporary backing file.
- `vhdl/arm_fpga_shared_stream_bridge.vhd`: shared-memory bridge RTL.
- `vhdl/tb_arm_fpga_shared_stream_bridge.vhd`: basic functional testbench.
- `vhdl/tb_arm_fpga_shared_stream_bridge_fast.vhd`: burst, wrap-around, and backpressure testbench.
- `docs/arm-fpga-shared-memory-stream.md`: protocol/register-map documentation for the bridge.
- `matlab/hft.prj`: MATLAB HDL Coder project metadata. At the moment this is mostly configuration, not a full MATLAB algorithm source tree.

## How The Repo Actually Works

The implemented prototype is not yet a full trading system. The concrete, working path today is:

1. `fast_data_feed` publishes synthetic FAST messages over TCP on port `9001`.
2. `fast_receiver` connects to that feed, decodes messages with mFAST, and prints them.
3. If `HFT_FPGA_MMIO_BASE` is set, `fast_receiver` also packs each decoded entry into a 128-bit frame and writes it into the ARM-to-FPGA TX ring.
4. The VHDL bridge exposes two ring buffers via MMIO:
   - TX ring: ARM -> FPGA
   - RX ring: FPGA -> ARM
5. Responses can be written by FPGA logic into the RX ring and read back by software.

## Build And Test Workflow

Use the `Makefile`. The intended flow is Dockerized so host toolchain differences do not matter.

Common commands:

- `make docker-image`
- `make cpp-test`
- `make vhdl-test`
- `make vhdl-test-fast`
- `make vhdl-wave`
- `make docker-shell`

C++ builds depend on a local checkout/install of `mFAST`, which the Make targets manage automatically through:

- `make mfast-clone`
- `make mfast-install`
- `make cpp-configure`
- `make cpp-build`

Notes:

- `cpp/CMakeLists.txt` auto-detects `../mFAST/install` if present.
- C++ tests build only `fpga_shared_stream_test`; the FAST sender/receiver are build targets, not covered by automated tests here.
- VHDL simulations generate VCDs under `vhdl/build/`.

## Runtime Knobs

`fast_receiver` enables the FPGA bridge only when these environment variables are provided:

- `HFT_FPGA_MMIO_BASE`: required physical base address
- `HFT_FPGA_MMIO_SPAN`: optional MMIO span, defaults to `0x1000`
- `HFT_FPGA_MMIO_DEV`: optional device path, defaults to `/dev/mem`

If `HFT_FPGA_MMIO_BASE` is unset, the receiver stays in software-only mode and just prints decoded messages.

## Cross-Language Invariants

When editing the shared stream interface, keep these aligned across C++, VHDL, tests, and docs:

- register offsets
- queue ownership rules
- ring full/empty semantics
- slot width in 32-bit words
- frame word ordering
- status bit meanings

Current frame packing from `fast_receiver.cpp` is:

- `word0`: sequence number
- `word1`: first 3 symbol bytes plus side code in the top byte
- `word2`: price scaled by `1e4`
- `word3`: quantity

The VHDL bridge stores slot lanes little-endian by 32-bit word index, and the testbenches assert that exact ordering.

## Important Pitfalls

### 1. RX base is conceptually dynamic

The HDL and docs define:

- `TX_BASE = 0x100`
- `RX_BASE = TX_BASE + DEPTH * SLOT_WORDS * 4`

`cpp/src/fpga_shared_stream.h` currently hardcodes:

- `kTxBase = 0x100`
- `kRxBase = 0x500`

That hardcoded `0x500` is correct only for the default layout (`DEPTH=64`, `SLOT_WORDS=4`). If you make queue geometry configurable in software, compute RX base dynamically from the discovered register values.

### 2. Ring capacity is `DEPTH - 1`

Both software and HDL use the standard single-producer/single-consumer convention where `next(head) == tail` means full. Do not accidentally treat all `DEPTH` slots as usable.

### 3. Publish order matters

For TX writes, software must write the slot payload before advancing `TX_HEAD`. For RX consumption, software must read the slot before advancing `RX_TAIL`.

### 4. MATLAB assets are thin right now

The repository mentions MATLAB HDL Coder heavily, but the tracked MATLAB content is currently just the project file. Do not assume generated VHDL in this repo comes directly from a complete checked-in MATLAB design.

## Editing Guidance

When you change the MMIO bridge contract:

- update `docs/arm-fpga-shared-memory-stream.md`
- update `cpp/src/fpga_shared_stream.h`
- update or extend `cpp/tests/fpga_shared_stream_test.cpp`
- update the VHDL bridge and whichever testbench covers the behavior

When you change FAST message structure:

- update `cpp/src/templates/SimpleMD.xml`
- expect generated mFAST outputs to change at build time
- verify both `fast_data_feed` and `fast_receiver`

When you change queue depth, slot width, or frame layout:

- re-check all hardcoded offsets in tests
- re-check the documented RX base computation
- re-check any assumptions in the C++ MMIO wrapper

## Suggested Agent Workflow

1. Read `README.md`, `Makefile`, and `docs/arm-fpga-shared-memory-stream.md`.
2. Inspect the relevant C++ and VHDL files before changing shared-interface behavior.
3. Prefer validating with the smallest matching test target:
   - C++ wrapper change: `make cpp-test`
   - RTL/register-map change: `make vhdl-test` and often `make vhdl-test-fast`
4. If changing the bridge protocol, update docs in the same patch.

## Current Testing Baseline

- C++: one focused test for the MMIO wrapper and ring behavior.
- VHDL: one basic testbench plus one higher-stress burst/wrap/backpressure testbench.
- No automated end-to-end test currently exercises `fast_data_feed` and `fast_receiver` together with real MMIO hardware.

## Good First Places To Look

- Shared-memory behavior: `docs/arm-fpga-shared-memory-stream.md`
- Software/FPGA interface code: `cpp/src/fpga_shared_stream.h`
- FAST decode and packing logic: `cpp/src/fast_receiver.cpp`
- Reference RTL behavior: `vhdl/arm_fpga_shared_stream_bridge.vhd`

---
> Source: [forestileao/hft-matlab-fpga](https://github.com/forestileao/hft-matlab-fpga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
