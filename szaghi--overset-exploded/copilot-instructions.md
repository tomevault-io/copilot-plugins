## overset-exploded

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Overset-Exploded is an overset post-processor for Xnavis/Xall that performs automatic block-splitting, load-balancing, and blocks-explosion. It reads overset grid (grd) and connectivity (icc) files, automatically splits blocks to balance computational workload across a specified number of processes, and generates per-block input files along with a proc.input file for process distribution.

## Build Commands

### Build with GNU Make
```bash
make
```
Executable will be created at `exe/overset-exploded`.

### Build with FoBiS.py
```bash
FoBiS.py build              # Release build (default: gnu mode)
FoBiS.py build -mode gnu-debug  # Debug build
```

### Clean Commands
```bash
make clean      # Remove objects and modules
make cleanall   # Remove objects, modules, and executables
make cleanobj   # Remove only objects
make cleanmod   # Remove only modules
make cleanexe   # Remove only executables
```

## Code Architecture

### Single-File Structure
The entire codebase is contained in a single Fortran file: `src/app/overset-exploded.F90` (~1485 lines). This is intentional for ease of compilation and distribution.

### Module Organization
The file contains three main components in order:

1. **`oe_block_object` module (lines 1-1118)**: Defines the `block_object` type representing a mesh block with:
   - Grid dimensions (Ni, Nj, Nk) and ghost cells (gc)
   - Node coordinates and connectivity data (icc, tcc, chimera arrays)
   - Block metadata (absolute index, group, body, processor assignment)
   - Splitting capabilities (split level, direction, parent tracking)
   - Methods for loading/saving block data, splitting blocks, and computing workload weights

2. **`oe_process_object` module (lines 1119-1164)**: Defines the `process_object` type representing a computational process with:
   - Assigned blocks list and workload tracking
   - Unbalance percentage relative to ideal workload
   - Methods for assigning blocks and initializing process data

3. **`overset_exploded` program (lines 1166-1485)**: Main program implementing the load-balancing algorithm:
   - Command-line argument parsing
   - File I/O for grd/icc formats
   - Load-balancing algorithm that iteratively assigns blocks to processes
   - Block splitting when necessary to achieve target unbalance threshold
   - Output generation (proc.input, optional imploded/exploded formats, Tecplot debug output)

### Data Flow
1. Load grid dimensions and connectivity from binary grd/icc files
2. Parse global rcc (connectivity) array into per-block tcc/chimera arrays
3. Compute block weights based on active cells
4. Create sorted blocks list (descending by workload)
5. Iteratively assign blocks to processes with minimum workload
6. If unbalance exceeds threshold, split largest unassigned block and retry
7. Save proc.input file and optionally save split block data

### Key Algorithms

**Block Weight Calculation**: Weight is based on the number of active cells (cells without boundary conditions). This represents computational workload since BC cells require less computation.

**Load Balancing Strategy**: Greedy algorithm that assigns the largest unassigned block to the process with minimum current workload. If assignment would exceed max_unbalance threshold, the block is split in the longest dimension while preserving multigrid level constraints.

**Block Splitting**: Blocks can be split recursively along i, j, or k directions. The split direction is chosen as the longest dimension that satisfies multigrid constraints (dimension must be divisible by 2^mgl). After splitting, chimera connectivity data is sanitized to remove invalid donor references.

### Boundary Condition Types
The code defines numerous BC constants (lines 22-65):
- Natural BCs: wall, symmetry, inflow, outflow, etc. (negative values: -1 to -19)
- Active cells: 0
- Chimera BCs: face-centered, cell-centered, adjacent, edge (positive values: 20-80)

### File Formats
- **GRD files**: Binary unformatted Fortran files containing block dimensions and node coordinates
- **ICC files**: Binary unformatted Fortran files containing cell-centered connectivity (icc) and global chimera data (rcc)
- **proc.input**: ASCII file listing block-to-process assignments
- **Imploded format**: Legacy format saving split blocks with original indexing
- **Exploded format**: New format saving individual block files with basename prefix

## Typical Usage

```bash
# Balance for 32 processes with default 1% max unbalance
overset-exploded -np 32

# Balance for 16 processes allowing 4% unbalance
overset-exploded -np 16 -max-unbalance 4

# Use non-default input files
overset-exploded -grd cc.02.grd -icc cc.02 -np 16

# Save proc.input with custom name and save split blocks in imploded format
overset-exploded -np 16 -proc-input proc.input-pes16 -save-imploded

# Enable Tecplot debug output
overset-exploded -np 16 -tec

# Preserve multigrid level 5 instead of default 4
overset-exploded -np 16 -mgl 5
```

## Development Notes

### Fortran Standards
- Code requires ISO Fortran 2008 standard
- Uses intrinsic module `iso_fortran_env` for kind parameters (R4P, R8P, I4P)
- Compiler flags: `-std=f2008 -fall-intrinsics` for gfortran

### Test Data
Test files are located in `src/tests/`:
- `cc.01.grd` - Sample grid file
- `cc.01` - Sample ICC file (assumes `.icc` extension is omitted)
- `proc.input` - Sample processor input file

### Memory Management
- Blocks are allocated dynamically based on file contents
- The global rcc array can be very large; it's deallocated after parsing into per-block data
- Split blocks create copies of parent data, so peak memory usage occurs during splitting operations

### String Utilities
The program includes custom string conversion functions (lines 1434-1484):
- `str()` - Convert integers/arrays to string with optional sign removal
- `strz()` - Convert integer to zero-padded string
These exist because the code is self-contained and doesn't depend on external libraries.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/szaghi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
