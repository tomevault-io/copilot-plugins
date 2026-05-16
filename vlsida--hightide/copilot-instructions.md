## hightide

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HighTide is a VLSI design benchmark suite built on OpenROAD-flow-scripts (ORFS). It ports open-source hardware designs to asap7/sky130hd/nangate45 technologies using the OpenROAD RTL-to-GDSII flow, serving as a benchmark suite for ML projects.

## Setup

Requires an Ubuntu machine with [Bazelisk](https://github.com/bazelbuild/bazelisk) (or Bazel 7.6.1+). No additional setup needed — Bazel fetches everything via `MODULE.bazel` and the pinned `bazel-orfs` submodule.

## Build Commands

```bash
# Build the full RTL-to-GDS flow for one design
bazel build //designs/asap7/lfsr:lfsr_final

# Build a design across all available platforms
bazel build //designs/asap7/minimax:minimax_final \
            //designs/nangate45/minimax:minimax_final \
            //designs/sky130hd/minimax:minimax_final

# Build all designs for a platform
bazel build //designs/asap7/...

# Build all designs across all platforms
bazel build //designs/...

# Build with dev RTL generation (requires submodule init + tools)
bazel build --define update_rtl=true //designs/asap7/lfsr:lfsr_final

# Build individual stages (target suffixes: _synth, _floorplan, _place, _cts, _grt, _route, _final)
bazel build //designs/asap7/lfsr:lfsr_synth
bazel build //designs/asap7/lfsr:lfsr_place
```

Note: `hightide_design()` exposes only the per-stage `orfs_flow` targets (`_synth` … `_final`) plus `_generate_abstract` / `_generate_metadata`. There is no aggregate `:<design>` target — use `:<design>_final` for the full flow.

## Architecture

### Key Relationships

- **`MODULE.bazel`** — Declares dependencies on `bazel-orfs` (via `git_override`) and configures the ORFS Docker image for tool extraction
- **`defs.bzl`** — `hightide_design()` macro wrapping `orfs_flow()` with common defaults (`GDS_ALLOW_EMPTY`, platform-to-PDK mapping)
- **`BUILD.bazel`** (root) — Defines `//:update_rtl` config setting and the `merge_yosys_share` target that bundles the yosys-slang plugin
- Each design has a `BUILD.bazel` calling `hightide_design()` with its parameters (utilization, density, SRAMs, etc.)
- RTL sources at `designs/src/<design>/BUILD.bazel` use `select()` to switch between release and dev RTL

### Design Configuration

Each design lives at `designs/<platform>/<design>/` and contains:
- **`BUILD.bazel`** — Calls `hightide_design()` with verilog_files, sources (SDC, LEFs, LIBs), arguments (utilization, density, etc.)
- **`constraint.sdc`** — Clock definitions and timing constraints
- **`pdn.tcl`** (optional) — Power delivery network configuration (layer stripes, pitches)
- **`io.tcl`** (optional) — Pin placement constraints

### RTL Source Management (`designs/src/`)

Design sources are git submodules under `designs/src/<design>/dev/repo/`.

Each `designs/src/<design>/BUILD.bazel` defines:
- `rtl_release` filegroup — pre-generated Verilog files
- `rtl_dev_gen` genrule — runs setup.sh/sv2v to generate from source
- `rtl` alias — uses `select({"//:update_rtl": ..., "//conditions:default": ...})` to pick

Dev mode requires: `git submodule update --init designs/src/<design>/dev/repo` before `bazel build --define update_rtl=true`.

### Platforms

| Platform | Node | Designs |
|----------|------|---------|
| asap7 | 7nm academic | coralnpu, gemmini, lfsr, litedram, minimax, sha3, vortex, liteeth_udp_usp_gth_sgmii, NVDLA (partitions a/c/m/o/p), bp_processor (bp_uno, bp_quad), cnn, floonoc, NyuziProcessor, snitch_cluster |
| nangate45 | 45nm | coralnpu, gemmini, lfsr, litedram, minimax, NyuziProcessor, sha3, liteeth_udp_usp_gth_sgmii, bp_processor (bp_uno, bp_quad), cnn |
| sky130hd | 130nm open | gemmini, lfsr, litedram, minimax, sha3, liteeth_udp_usp_gth_sgmii, cnn |

#### Build status (as of 2026-05-11)

Designs reaching `_final` (cached on remote build cache):
- **asap7**: coralnpu, gemmini, lfsr, litedram, minimax, sha3, vortex, liteeth_udp_usp_gth_sgmii, NVDLA partitions a/m/o
- **nangate45**: coralnpu, gemmini, lfsr, litedram, minimax, NyuziProcessor, sha3, liteeth_udp_usp_gth_sgmii
- **sky130hd**: gemmini, lfsr, litedram, minimax, sha3, liteeth_udp_usp_gth_sgmii, NVDLA partitions a/m/o/p

Not yet finishing (not cached):
- **asap7**: cnn, floonoc, NyuziProcessor, snitch_cluster, bp_processor (bp_uno, bp_quad), NVDLA partitions c, p
- **nangate45**: cnn, bp_processor (bp_uno, bp_quad)
- **sky130hd**: cnn, NVDLA partition c

`sky130hd/NVDLA/partition_c` global placement plateaus at overflow ~0.31 (target 0.10) — 84 macros at sky130hd's coarse pitches create local density hot spots near macro pin clusters that the placer can't smooth out. Loosening `CORE_UTILIZATION` / `PLACE_DENSITY_LB_ADDON` / `MACRO_PLACE_HALO` to match working partitions doesn't fix it. Likely needs a manual `macros.tcl` to spread the 84 SRAMs into a regular grid.

Use `tools/fetch_cache.sh` to pull cached `_final` results from the remote cache; designs marked NOT CACHED there are the not-yet-finishing set above.

### Output Directories

Build artifacts go to `bazel-bin/designs/<platform>/<design>/`. Key outputs:
- `results/<platform>/<design>/base/6_final.{odb,v,sdc,spef}` — Final ODB and netlist (and `6_final.gds` when `GDS_ALLOW_EMPTY` is not suppressing GDS write)
- `results/<platform>/<design>/base/<N>_<stage>.{odb,sdc,...}` — Per-stage outputs (1_synth, 2_floorplan, 3_place, 4_cts, 5_1_grt, 5_2_route, 6_final)
- `reports/<platform>/<design>/base/` — QoR and OpenROAD-generated `.webp.png` heatmaps/placement images written by ORFS's own report stage
- `logs/<platform>/<design>/base/` — Per-stage logs

### FakeRAM

Designs with embedded memories use FakeRAM (LEF-only, no GDS). Controlled by `GDS_ALLOW_EMPTY = fakeram.*` set as a default in `defs.bzl`.

SRAM LEF/LIB files are organized per-platform:
- `designs/<platform>/NyuziProcessor/sram/{lef,lib}/` — NyuziProcessor memories
- `designs/<platform>/liteeth/sram/{lef,lib}/` — liteeth memories (`fakeram_1rw1r_64w64d` + `fakeram_1rw1r_64w1024d`)
- `designs/asap7/bp_processor/sram/{lef,lib}/` — bp_processor memories
- `designs/src/cnn/fakeram_*.{lef,lib}` — CNN asap7 memories (shared with src dir)
- `designs/{nangate45,sky130hd}/cnn/sram/{lef,lib}/` — CNN per-platform synthetic FakeRAMs (regenerable via `designs/src/cnn/dev/gen_fakeram_{nangate45,sky130hd}.py`)

## Reporting cell counts

When reporting or comparing cell counts across designs or platforms, **exclude fill cells, tap cells, and antenna cells**. They are physical-only / manufacturing-rule cells, not part of the design's logic. Their counts vary wildly across platforms (sky130hd uses ~17× more tap cells than nangate45 due to a stricter max-tap-distance rule, and lower `CORE_UTILIZATION` inflates filler counts proportionally), so including them obscures real differences.

Use the per-class fields in `6_report.json` (e.g. `class:sequential_cell`, `class:multi_input_combinational_cell`, `class:inverter`, `class:clock_buffer`, `class:timing_repair_buffer`) — not the top-level `instance__count` — when comparing designs.

## Known OpenROAD / yosys-slang bug workarounds

Designs in this repo carry workarounds for upstream tool bugs. Update this table whenever a new workaround lands, and remove rows once the upstream bug is fixed and the workaround can be reverted.

| Bug | Affected designs | Workaround | First commit | Issue |
|-----|------------------|------------|--------------|-------|
| **CTS-0105** false skip — yosys hierarchical synthesis output port buffers arrive in ODB with `dbSourceType::TIMING` instead of `NETLIST`; CTS skips them as pre-existing clock buffers and leaves the clock net unbuffered | asap7 NyuziProcessor; asap7/nangate45/sky130hd `bp_quad`, `bp_uno` | `PRE_CTS_TCL` script resets affected buffers' `dbSourceType` from `TIMING` → `NETLIST` before CTS runs | `81b0ed4b` | [OpenROAD#10177](https://github.com/The-OpenROAD-Project/OpenROAD/issues/10177) |
| **MPL-0040** — `rtl_macro_placer` annealing failure on certain macro clusters | asap7 `bp_quad` (pipe_fma cluster), asap7 `cnn` | Hand-place fakeram macros in left-edge columns at FIRM status via `macros.tcl` so RTLMP only sees pre-placed macros; for cnn also drop `CORE_UTILIZATION` 65→60 | `09542e19` | [OpenROAD#9985](https://github.com/The-OpenROAD-Project/OpenROAD/issues/9985) |
| **ODB-1200** in `repair_timing` — `InsertBufferBeforeLoads` iterates a stale load list and aborts the flow with `Load pin '...' is not connected to net '...'`. Triggered by the resizer's `SplitLoadMove` step in CTS-time repair_timing | asap7 `liteeth_udp_usp_gth_sgmii`; sky130hd `NyuziProcessor`; asap7 `bp_quad`; asap7 `gemmini` (this PR) | Most designs: `SKIP_CTS_REPAIR_TIMING = 1` (skips the whole repair pass; route still does its own hold-repair). Gemmini: drop `split_load` from `SETUP_MOVE_SEQUENCE` (`"unbuffer,sizeup,swap,buffer,clone"`) — keeps the rest of repair_timing working | `87829fdb` | [HighTide#75](https://github.com/VLSIDA/HighTide/issues/75) |
| **DPL-0036** in CTS-internal `detailed_placement` — `cts.tcl` builds its own `dpl_args` ignoring `DETAIL_PLACEMENT_ARGS`, so CTS-inserted leaf clock buffers can land too far from a legal stdcell row | asap7 `snitch_cluster` | `PRE_CTS_TCL` wraps `detailed_placement` to inject `-max_displacement {2000 400}` for the CTS call | `0af20020` | None (ORFS layering issue; no upstream issue filed yet) |
| **yosys-slang** phantom 1'x drivers on interface-array modport ports — extra driver wins at `opt_clean`, real driver silently dropped | asap7 `vortex` | Bumped `yosys-slang` 4e53d77 → eabdfd1 (vendored via `MODULE.bazel`) | `cb488bea` | [yosys-slang#304](https://github.com/povik/yosys-slang/issues/304) |
| **`repair_timing` non-convergence** — repair loop spends iterations on the same RTL-bounded endpoint without making progress. Not a bug per se, but a flow pathology when the clock target is too tight for the design | asap7 `snitch_cluster`, asap7 `litedram` | `SKIP_INCREMENTAL_REPAIR = 1` to skip post-GRT `repair_timing` | `39ca8670` | None |
| **LiteX GENSDRPHY `input sdram_dq`** — `litedram/gen.py` declares the bidirectional SDR DQ bus as a plain `input` port (Lattice-platform convention; the tristate buffer lives at the IOB), but the same module instantiates 16 `TRELLIS_IO` cells that internally drive the port via OE-gated logic. yosys' `check -assert -mapped` sees the conflict and aborts synth on all 3 platforms | asap7/nangate45/sky130hd `litedram` | Patch `litedram_core.v` to make `sdram_dq` an `inout` port (`patch/litedram_core.patch`). Combined with `SYNTH_HIERARCHICAL = 1` so abc keeps each `TRELLIS_IO` as a hierarchy boundary | (this PR) | None (LiteX `gen.py:870` carries a `# FIXME: Allow other Vendors.` note) |

### Useful ORFS env vars for these workarounds

- **`PRE_CTS_TCL`** / **`PRE_GRT_TCL`** / etc — point at a Tcl script that runs before the named stage. Used for the CTS-0105 reset and DPL-0036 displacement injection.
- **`SETUP_MOVE_SEQUENCE`** — comma-separated list of moves passed to `repair_timing -sequence`. Default sequence is `unbuffer,sizeup,swap,buffer,clone,split_load`. Drop `split_load` to dodge ODB-1200 while keeping the rest of repair_timing active.
- **`SKIP_CTS_REPAIR_TIMING = 1`** — skip the entire `repair_timing_helper` call inside `cts.tcl`. Heavier hammer than dropping a single move; route's own hold-repair still runs.
- **`SKIP_INCREMENTAL_REPAIR = 1`** — skip post-GRT `repair_timing`. Use when the loop never converges (RTL-bounded critical paths).
- **`SKIP_LAST_GASP = 1`** — skip the final repair_timing pass.

These go into `arguments = { ... }` of `hightide_design()`.

## Shared Machine

- This is a shared multi-user machine. When checking process status (e.g., `ps aux | grep openroad`), always filter to the current user's processes or check for bazel sandbox paths (`external/bazel-orfs`) to avoid confusing other users' builds with ours.

## Bazel Cache

- **NEVER** run `bazel clean --expunge` or delete the entire disk cache (`~/.cache/bazel-disk-cache`). Synthesis takes hours and clearing the cache forces a full rebuild of all designs.
- To invalidate a single design's cached results, use targeted approaches:
  - Change an argument in the design's BUILD.bazel (forces re-run of affected stages)
  - Use `bazel build --strategy=<target>=local` to bypass cache for one target
- The remote cache (`--remote_cache` in `.bazelrc`) is shared and read-only by default. Never delete or corrupt it.
- If you suspect a stale cache, verify by checking the process count in bazel output: `N processwrapper-sandbox` means N actions actually ran; `N internal` means cached.

---
> Source: [VLSIDA/HighTide](https://github.com/VLSIDA/HighTide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
