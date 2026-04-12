## epc-gen2-simulation-progress2

> This repository contains the Verilog source code for an EPC Class-1 Generation-2 RFID Tag. We are currently emulating the logic on a Digilent Arty S7 FPGA (5 MHz system clock) to verify the physical layer using a ThingMagic reader before finalizing the design.

# EPC Gen2 RFID Tag - FPGA Emulation

## Project Overview
This repository contains the Verilog source code for an EPC Class-1 Generation-2 RFID Tag. We are currently emulating the logic on a Digilent Arty S7 FPGA (5 MHz system clock) to verify the physical layer using a ThingMagic reader before finalizing the design.

## ⚠️ STRICT HARDWARE CONSTRAINTS (ASIC Tape-Out)
The core logic of this tag is destined for a **Cadence TSMC 0.18 µm ASIC tape-out**. 
* **DO NOT MODIFY:** Core ASIC files such as `cmd_proc.v`, `crg.v`, and internal gated clocks are completely frozen.
* **MODIFIABLE:** Only the FPGA-specific digital wrapper (`pie_decoder.v`) and top-level routing (`rx.v`, `bb_proc.v`, `fpga_top.v`) can be edited.

## Current Debugging Status
We are fixing the handoff between the `pie_decoder` wrapper and the `cmd_proc`. 
1. **The Bug:** `fs_detector` successfully reaches `S_DONE`, but the `pie_decoder` gets stuck outputting repeating `1`s. `package_complete` never fires, leaving the state machine stuck in `Ready (0x0)`.
2. **Pending Fixes:**
   * Remove duplicate/dead code in `pie_decoder.v`.
   * Fix the FSM in `pie_decoder.v` so it properly resets and transitions on the falling edge of the RF pulse.
   * **Routing Fix:** Route the *raw* `pie_code` signal directly to the `crg` module to drive `rst_for_new_package` correctly (it is currently incorrectly using the decoded `~pie_code`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SaminRules)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SaminRules)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
