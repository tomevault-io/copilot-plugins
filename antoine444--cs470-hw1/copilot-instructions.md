## cs470-hw1

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EPFL CS-470 (Advanced Computer Architecture) Homework 1: a cycle-by-cycle simulator of an out-of-order processor called OoO470. It runs a minimal RISC-V instruction subset with register renaming, reservation stations, and exception handling. The simulator reads instructions from JSON, simulates cycle-by-cycle, and dumps full CPU state per cycle as JSON.

## Build and Run

The student must create two scripts:
- `build.sh` — compiles/builds the solution (can be empty for interpreted languages; can use `apt` to install dependencies)
- `run.sh <input.json> <output.json>` — runs the simulator

Key commands:
- `./runall.sh` — builds and runs on all 9 test cases (produces `user_output.json` in each test dir)
- `./testall.sh` — builds, runs, and validates against reference outputs
- `python compare.py <user_output.json> -r <reference_output.json>` — compare a single test case
- Grading environment: Ubuntu 22.04 container with C17/C++17, Python 3.10, Go 1.20, Rust 1.67, Java 17, Scala 3.2.2, Node 18.14

## Test Cases

9 test cases in `given_tests/01/` through `given_tests/09/`, each with `desc.txt`, `input.json`, `output.json`. Tests cover: data dependencies (RAW, WAR, WAW), exception handling (single/multiple/last instruction), long dependency chains, and no-dependency cases.

## Instruction Set

Six instructions, all arithmetic on 32 64-bit registers (x0-x31):
- `add dest, opA, opB` — dest = opA + opB
- `addi dest, opA, imm` — dest = opA + sign-extended immediate
- `sub dest, opA, opB` — dest = opA - opB
- `mulu dest, opA, opB` — dest = unsigned(opA) * unsigned(opB)
- `divu dest, opA, opB` — dest = unsigned(opA) / unsigned(opB) (exception if opB=0)
- `remu dest, opA, opB` — dest = unsigned(opA) % unsigned(opB) (exception if opB=0)

PC starts at 0 and increments by 1 per instruction. For `addi`, operand B is the sign-extended immediate.

## Pipeline Architecture

Six pipeline stages (numbered 0-5 in the pipeline diagram):

### Stage 0-1: Fetch and Decode (F&D)
- Fetches up to 4 instructions per cycle from program memory
- PC resets to 0 each cycle, then increments by number of instructions actually fetched
- Stalls (applies backpressure) if Rename and Dispatch cannot accept instructions
- On exception detection from Commit: PC set to 0x10000, Decoded Instruction Register cleared

### Stage 1-2: Rename and Dispatch (R&D)
- Processes all 4 instructions atomically (all or none)
- Checks for enough free physical registers, Active List slots, and Integer Queue slots
- Renames destination registers: allocates physical register from Free List, updates Register Map Table
- Reads operand state: ready from physical register file, forwarding path, or marks as not-ready (via Busy Bit Table)
- Allocates entries in both Active List and Integer Queue
- Observes forwarding paths and updates physical register file and Busy Bit Table

### Stage 2-3: Issue (I)
- Scans Integer Queue, issues up to 4 ready instructions to ALUs per cycle
- Priority: oldest instructions first (smallest PC)
- Can issue instructions whose operands are ready OR will be ready via forwarding path this cycle
- Issued instructions removed from Integer Queue next cycle

### Stage 3-4: Execution (ALUs)
- 4 ALUs, each with 2-cycle latency
- Results broadcast on forwarding paths on the second ALU cycle (pipeline register 4)
- Forwarding paths update Integer Queue entries, physical register file, and Busy Bit Table

### Stage 4-5: Commit (C)
- Scans Active List in program order, retires up to 4 completed instructions per cycle
- Stops at: (a) 4 instructions retired, (b) incomplete instruction, or (c) completed instruction with exception
- Retired instructions' old physical registers returned to Free List
- On exception: enters Exception Mode

### Exception Recovery (Exception Mode)
- Sets Exception PC (ePC) to the faulting instruction's PC, sets Exception Flag
- Halts fetch/decode, clears Integer Queue and Execution stage
- Rolls back Active List from bottom (newest) in reverse program order, up to 4 per cycle: restores Register Map Table, Free List, Busy Bit Table
- Exits Exception Mode when Active List is empty
- PC set to 0x10000, simulation stops

## Key Implementation Details

### Resource Sizes
- 64 physical registers (p0-p63), initialized to 0
- 32 architectural registers (x0-x31), initially mapped to p0-p31
- Free List initially contains p32-p63 (FIFO)
- Active List: max 32 entries
- Integer Queue: max 32 entries
- Busy Bit Table: 64 booleans, all initially false

### Timing Model (propagate/latch)
- Use a **propagate** then **latch** approach: copy current state, compute next state on copies using combinational logic, then latch (swap) at end of cycle
- All data structures are asynchronous-read (combinational read path — reads and writes in same cycle are visible)
- Updates to Active List, Integer Queue, and Free List within a cycle are visible to other units in the same cycle (combinational path between write and read ports)
- State is dumped at the **beginning** of each cycle (after latch, before propagate)

### Free List Ordering
- Normal commit: physical registers freed in program order (earlier instruction's old reg freed first)
- Exception recovery: physical registers freed in reverse program order (later instruction's new reg freed first)

### Simulation Loop (pseudocode)
```
parseInstructions()
dumpState()  // dump reset state
while (hasInstructions() or activeListNotEmpty()):
    propagate()  // combinational logic on copies
    latch()      // swap in next state
    dumpState()  // dump beginning-of-cycle state
saveLog()
```

## Output Format

JSON array of cycle state objects. The comparator (`compare.py`) treats FreeList and BusyBitTable as sets (order-independent) and IntegerQueue as a set of entries. ActiveList order matters. Fields where value is "don't care" (e.g., OpAValue when OpAIsReady is false) are not checked by the comparator.

## Visualization

Open `visualize.html` in a browser to load and inspect output JSON files cycle-by-cycle.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Antoine444) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
