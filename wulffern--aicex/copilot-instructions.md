## aicex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<<<<<<< HEAD
## What This Repository Is

A monolithic workspace for analog integrated circuit (IC) design, containing both the open-source EDA tools and the IP blocks that use them. Dependencies (tool repos and IP blocks) are pinned in `config.yaml` and managed via `cicconf` — **never add subdirectories directly to git**.

## Commands

### Dependency Management
```bash
cicconf clone       # Clone all repos listed in config.yaml
cicconf update      # Update pinned SHAs
cicconf pull        # Pull latest for all repos
cicconf status      # Show status across all repos
cicconf newip --project <proj> --technology <tech> <name>  # Create new IP block
```

### Tool Development (install editable)
```bash
cd cicconf && pip install -e .
cd cicsim  && pip install -e .
cd cicpy   && pip install -e .
=======
## Project Overview

**AICEX** (Analog Integrated Circuit Ecosystem) is a monorepo managing multiple analog IC design IPs. It contains Python CLI tools (`cicsim`, `cicpy`, `cicconf`, `cicspi`) and ~70+ individual IC design projects targeting SkyWater 130nm PDK.

## Core Python Tools

### Install (development mode)
```bash
pip install -e cicconf/
pip install -e cicspi/
pip install -e cicpy/
pip install -e cicsim/
>>>>>>> 3786580bf442aa2b67f320d2bd9df41e8b4d684f
```

### Testing
```bash
<<<<<<< HEAD
# cicsim unit tests (no EDA tools required)
cd cicsim && make unit_test

# cicsim integration tests (requires ngspice)
cd cicsim && make test

# cicpy integration tests
cd cicpy && make test
# or a single suite, e.g.:
cd cicpy/tests/transpile && make test
```

### Releasing a tool to PyPI
```bash
cd <tool> && make build && make upload
```

### IP-level simulation
```bash
cd <ip_name>/sim
cicsim run <Testbench>.spi --corners Tt Th Tl
cicsim wave                          # open waveform viewer
cicsim results                       # tabular summary
cicsim summary                       # markdown report with spec checking
```

### IP-level layout/verification
```bash
cd <ip_name>/work
make drc cdl lvs OPT='--short'
make gds
```

### Batch top-level
```bash
make lelo   # Runs drc/cdl/lvs across all lelo_gr0* blocks in parallel
```

## Architecture

### Repository Layout

```
config.yaml          # Pinned dependency list (cicconf reads this)
Makefile             # Top-level batch targets (e.g. make lelo)
cicconf/             # Dependency & clone manager
cicsim/              # Simulation orchestrator + waveform viewer
cicpy/               # Transpiler (.cic JSON → SPICE/SKILL/Verilog/Magic/SVG)
ciccreator/          # C++ generator for SAR ADC layouts (.cic.gz files)
cicspi/              # SPICE netlist parser (used by cicpy)
cpdk/                # Carsten's PDK: XSchem symbols, design rules
tech_sky130A/        # Technology library for Skywater 130A (Tiny Tapeout)
tech_sky130B/        # Technology library for Skywater 130B
<proj>_<name>_<tech>/  # IP blocks (one directory per block)
```

### IP Naming Convention
`<project>_<block>_<technology>` — e.g. `lelo_gr01_sky130a`, `jnw_atr_sky130a`, `cnr_ota_sky130nm`.

### IP Directory Structure
```
<ip>/
├── design/      # Schematics, RTL sources
├── sim/         # Testbenches, cicsim YAML configs
├── work/        # Layout, DRC/LVS outputs (generated)
├── cic/         # ciccreator source files (if applicable)
├── docs/        # Documentation, images
├── tech -> ../tech_*/   # Symlink to technology library
├── Makefile     # Minimal: include tech/make/main.make
├── config.yaml  # IP-specific tool config
├── info.yaml    # Metadata (description, authors)
└── README.md
```

### Design Flow
1. **Schematic / Netlist** — hand-written SPICE (`.spi`) or ciccreator → `.cic.gz` → `cicpy transpile` → SPICE/Verilog/SKILL
2. **Simulation** — `cicsim run` orchestrates ngspice across corner matrix (Tt/Tl/Th × Vt/Vh/Vl), stores results as pandas DataFrames, validates against specs in YAML
3. **Layout** — hand-drawn XSchem `.sch` or `cicpy spi2mag` auto place-and-route → Magic `.mag`
4. **Verification** — DRC, LVS, CDL via `make` targets driven by `tech/make/main.make`
5. **Export** — GDS generation; `GDS2glTF` for 3D visualization

### Key Tool Source Files
| Tool | Entry point | Notable modules |
|------|-------------|-----------------|
| cicconf | `cicconf/entry.py` | `config.py`, `cmdip.py` |
| cicsim | `cicsim/cicsim.py` | `cmdrunng.py` (ngspice runner), `cmdwave_pg.py` (PyQtGraph viewer), `ngraw.py` (binary raw parser), `spec.py`, `pivot.py` |
| cicpy | `src/cicpy/cic.py` | `core/` (data model), `printer/` (output generators), `eda/` (Magic/Xschem integration) |

## Rules
- **Never add IP or tool folders directly to git** — use `cicconf` to manage them.
- **Always use [Click](https://click.palletsprojects.com/) for CLI argument parsing** in all Python tools.
- **Document your work** — each IP has a `README.md`; tools use docstrings and `--help`.
- Simulation corners are temperature × voltage pairs: `Tt/Tl/Th` × `Vt/Vh/Vl`.
- IPs use a shared `Makefile` pattern: a minimal local file that does `include tech/make/main.make`.
- CI container: `wulffern/aicex:26.04_latest` (has ngspice, Magic, Xschem, Python env).
=======
# cicsim unit tests
cd cicsim && python3 -m unittest discover -s tests/unittests/ -p 'test_*.py' -v

# cicsim full test (requires ngspice)
cd cicsim && make test

# Single test file
cd cicsim && python3 -m unittest tests/unittests/test_<module>.py
```

### Linting
```bash
cd cicsim && ruff check cicsim/
cd cicpy  && ruff check src/
```

## IP Repository Management

IPs are cloned/versioned via `cicconf` using `config.yaml`:
```bash
cicconf clone          # Clone all IPs listed in config.yaml
cicconf update         # Update IPs to latest revisions
```

Each IP lives in a directory named `{project}_{name}_{technology}` (e.g., `lelo_temp_sky130a`).

## IC Design Workflow (per IP)

Each IP follows this directory structure:
- `design/` — Xschem schematics (`.sch`)
- `work/` — Makefile-driven layout/DRC/LVS/GDS
- `sim/` — ngspice testbenches, `cicsim.yaml` configs
- `simrtl/` — SystemVerilog testbenches
- `rtl/` — Verilog/SystemVerilog RTL
- `py/` — Python layout hooks (`<CELL>.py`) and post-processing models
- `docs/` — Jekyll markdown, auto-published to GitHub Pages

### Layout flow (Xschem → Magic → GDS)
```bash
cd <ip>/work/
make mag CELL=<CELLNAME>   # Schematic → Magic layout via cicpy
make drc CELL=<CELLNAME>   # DRC with Magic
make lvs CELL=<CELLNAME>   # LVS with netgen
make gds CELL=<CELLNAME>   # Export GDS
```

cicpy layout hooks live in `py/<CELL>.py`. M1 is reserved for power; M2/M3/M4 for signal routing.

### Simulation flow
```bash
cd <ip>/sim/
cicsim run tb_<name>/tb.ngspice          # Single run
cicsim run tb_<name>/tb.ngspice --threads 4  # Multi-threaded corner runs
```

`cicsim.yaml` in each testbench directory controls corners (temperature/voltage combos), spec checking, and result summaries (Markdown output).

## Architecture: How the Tools Interact

```
config.yaml
    └─ cicconf clone → individual IP git repos
                            ├─ Xschem .sch
                            │       ↓ cicpy transpile
                            ├─ Magic .mag → netgen LVS / Magic DRC → GDS
                            └─ ngspice testbenches
                                    ↓ cicsim run
                               .raw waveforms → pandas → Markdown spec report
```

## CI/CD

GitHub Actions (`.github/workflows/`) run GDS/DRC/LVS/DOCS pipelines using Docker image `wulffern/aicex:26.04_latest`. Workflows delegate to shared reusable workflows in the `jnw-actions` repo. The `PDK_ROOT` env var must point to the SkyWater PDK (typically `/opt/pdks/sky130A` in CI).

>>>>>>> 3786580bf442aa2b67f320d2bd9df41e8b4d684f

---
> Source: [wulffern/aicex](https://github.com/wulffern/aicex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
