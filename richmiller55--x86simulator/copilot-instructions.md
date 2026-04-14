## x86simulator

> This project is an simulator built in C++17, designed to execute instructions and simulate the core components of a computer system. It is integrated with a PostgreSQL database and features a text-based user interface built with ncurses. It strives to be a ISA simulator with a IR stucture . currently limited coverage is implemented for Intel 86 and even less coverage for ARM, Plans for future support for powerPC RISC-V, The simulator will support pipelining

# Simulator: GEMINI Project

## Project Overview

This project is an simulator built in C++17, designed to execute instructions and simulate the core components of a computer system. It is integrated with a PostgreSQL database and features a text-based user interface built with ncurses. It strives to be a ISA simulator with a IR stucture . currently limited coverage is implemented for Intel 86 and even less coverage for ARM, Plans for future support for powerPC RISC-V, The simulator will support pipelining

### Key Technologies

*   **Language:** C++17
*   **Compiler:** `g++`
*   **Build System:** `Make`
*   **Database:** PostgreSQL (using `libpqxx`)
*   **UI:** `ncurses`
*   **json:** `nlohmann/json`
*   **serialization:** `cereal`
*   **Test Framework:** `gtest`

### Architecture

The simulator is structured around these key classes:

*   `SystemBus`: Reads a config file. system_bus.json that sets process up
*   `ISimulator`: An interface for a generic instruction set simulator
*   `DatabaseManager`: Handles all interactions with the PostgreSQL database.
*   `RegisterMap`: Manages the state of the CPU's registers.
*   `Memory`: Simulates the computer's memory for instruction and data storage.
*   `UIManager`: Manages the ncurses-based text user interface.
*   `Decoder`: Responsible for decoding x86 instructions for execution.
*   `IROpcode`: Generic/Scalar Operations


## Building and Running

### Building the Simulator

To compile the source files and create the `x86simulator` executable, use the following `make` command:

```bash
make -f Makefile.mk
```

### Running Tests

To run the test suite, use the following command:

```bash
make -f Makefile.mk test
```


{
"Coding Style":
Variable Naming":  "Use snake_case for all variable names.",
"Member Variables": "Member variables will have a trailing underscore (e.g., decoder_).",
"Filenames": 	    "Use snake_case.cpp for all new source files.",
"General Style": "The code uses header guards and separates concerns into different classes for clarity."
"Database Integration": {
"Connection String": "The connection to the PostgreSQL database is configured via the DB_CONN_STR environment variable."
}
"Testing": {
"Test Framework": "The project uses gtest".

Target directory: `/home/rmiller/src/cpp/simulators/x86/`

## todays todo: flesh out ARM support
resolve failing tests,
add firstPass secondPass functions to Arm_simulator_core

## continuing Work: Multi-ISA Support

This section outlines a design for a multi-ISA (Instruction Set Architecture) simulator. The core idea is to create a generalized simulation engine by abstracting away the specifics of any single architecture like x86, ARM, or RISC-V. This is achieved through a powerful Intermediate Representation (IR) and a modular design.

### Summary of Design Goals

The final design, as you've hinted, will be much cleaner. The VPU, ALU, and FPU will be components owned by
  the ExecuteStage of the pipeline. The ExecuteStage will receive instructions from the DecodeStage and
  dispatch them to the correct functional unit. These units will interact with a generic RegisterMap via the
  ISimulator interface, configured by the Architecture class for the specific ISA. The Scoreboard will also
  use this Architecture information for hazard detection.
  

Th### Specific TODO List
  managing the flow of instructions through different stages. It will contain instances of the various pipeline stages and coordinate their operations in a step-by-step fashion. A single tick() method could be the entry point for advancing the entire pipeline by one clock cycle.Stage classes (e.g., FetchStage, DecodeStage, ExecuteStage):  The ExecuteStage can contain a map or other data structure to hold instances of specialized functional unit helper classes. For example, std::map<FunctionalUnitType, IFunctionalUnit*>.IFunctionalUnit interface: A base interface for all functional units. It should include:execute(Instruction& instruction): Performs the necessary computation on the instruction.is_busy(): A method for the scoreboarding system to check for structural hazards.latency(): Returns the number of cycles required to complete an operation, which is critical for scoreboarding.Example functional unit classes:ALU : public IFunctionalUnit: For integer operations.FPU : public IFunctionalUnit: For floating-point operations.VPU : public IFunctionalUnit: For vector operations. Pipeline scoreboarding and hazard detection A scoreboard is essential for tracking data dependencies and managing the pipeline for out-of-order execution. Scoreboard class: This helper class can live within the Pipeline class. It manages a table of all in-flight instructions and their register dependencies.Register tracking: The scoreboard needs to track which registers are being written to by instructions currently in the pipeline.It can use a std::vector<InstructionID> or similar to track the instruction ID that will produce the value for a given register.Hazard types: The scoreboard will detect and track different kinds of hazards:RAW (Read-After-Write) hazard: An instruction tries to read a register that a prior instruction is still writing. The scoreboard can stall the dependent instruction until the data is available.WAW (Write-After-Write) hazard: Two instructions attempt to write to the same register. This is a crucial hazard to manage, as the later instruction must write last. Register renaming is a more advanced technique to eliminate this.WAR (Write-After-Read) hazard: An instruction tries to write to a register that a later instruction has already read. Out-of-order pipelines must handle this carefully.Pipeline wins and losses scoring system: This adds the educational component you wanted.You can integrate a ScoreCounter class that increments counters for different events."Win": Successfully forwarding data without a stall, indicating good pipeline performance."Loss": A pipeline stall occurs due to a data or structural hazard.Metrics: Track a range of metrics: cycles_stalled_for_raw, total_bubbles_inserted, functional_unit_busy_cycles, etc.This provides concrete data to visualize and analyze the efficiency of different pipeline configurations. Pipeline latches Latches (or pipeline registers) are the data structures that hold an instruction and its state as it moves between pipeline stages. Latch template class: A template<typename T> class can be used to pass data between stages.Double-buffering: A latch should have a current state and a next state. This prevents race conditions and ensures all stages see a consistent view of the data from the previous clock cycle. A commit() method can transfer the next state to the current state at the end of every clock cycle.Example instantiation:Latch<Instruction> if_id_latch;: For data between the fetch and decode stages.Latch<Instruction> id_exe_latch;: For data between the decode and execute stages. Class diagram overview \(\text{Pipeline}\longrightarrow \text{FetchStage}\longrightarrow \text{Latch<Instruction>}\longrightarrow \text{DecodeStage}\longrightarrow \text{Latch<Instruction>}\longrightarrow \text{ExecuteStage}\longrightarrow \text{Scoreboard}\)Helper dependencies: ExecuteStage contains IFunctionalUnit* (like FPU, VPU).Pipeline contains Scoreboard and ScoreCounter. This design separates concerns, making it easier to implement and reason about each component. By adding different IFunctionalUnit implementations and modifying the Scoreboard logic, you can explore the architectural differences between ARM (simpler, in-order pipelines) and high-end Intel (complex, out-of-order pipelines with register renaming).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/richmiller55)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/richmiller55)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
