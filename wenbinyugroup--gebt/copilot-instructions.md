## gebt

> GEBT — a Fortran solver for Geometrically Exact Beam Theory (v4.8). Mixed-formulation intrinsic beam theory compatible with VABS cross-sectional properties.

# AGENTS.md

## What This Is

GEBT — a Fortran solver for Geometrically Exact Beam Theory (v4.8). Mixed-formulation intrinsic beam theory compatible with VABS cross-sectional properties.

## Language and Build

- **Source language**: Fortran — fixed-form `.f` (legacy libraries) and free-form `.f90` (application code). `pyproject.toml` is for Sphinx doc tooling only, not the main application.
- **Compiler**: gfortran (primary, cross-platform). Intel `ifx` also supported as fallback.
- **Build (CMake, recommended)**:
  - Windows: `.\tools\build_msvc.bat` (requires gfortran on PATH + MSYS2 OpenBLAS/ARPACK)
  - Linux: `cmake -B build . && cmake --build build`
  - CMake auto-detects system libraries via `find_library`; falls back to bundled sources if not found.
  - Link order: arpack → OpenBLAS → gomp (static libs, order matters).
  - gfortran builds use `-static-libgfortran -static-libgcc` on Windows to avoid a UCRT heap conflict with `TRIM()` temporaries.
- **Windows MSYS2 dependencies**: `pacman -S mingw-w64-ucrt-x86_64-openblas mingw-w64-ucrt-x86_64-arpack`
- **Intel ifx build (fallback)**: `.\tools\build_ifx.bat` (requires Intel oneAPI command prompt)
- **Legacy Makefile**: `cd tools && make install` (gfortran, reads `tools/gebtMake` for source list).

## Architecture

Entry point: `src/main.f90` → calls `Input` (IO module) → `Analysis` → `Output` (IO module).

Module dependency chain (approximate):
```
GlobalDataFun → TimeFunction, PrescribedCondition → InternalData →
Preprocess → Element → Member → System → Solve, EigenSolve → Analysis → IO
```

Bundled libraries in `src/`: MA28 (sparse linear solver), BLAS, LAPACK. ARPACK replaced by system arpack-ng when available.

## Fortran Coding Conventions

Documented in `src/GlobalDataFun.f90:1-50`. Key rules:
- Global constants, Fortran keywords, and intrinsic functions: **UPPERCASE**
- User-defined procedures: **CamelCase** (e.g. `MySubroutine`)
- All other variables: **lowercase_with_underscores**
- Always use `IMPLICIT NONE`
- Never use `STOP` in subroutines — return error message instead
- Use `USE ModuleName, ONLY: ...` for imports
- `PRIVATE` by default; explicitly `PUBLIC` only what is needed
- Include a data dictionary in program unit headers

## Running / Testing

- **No automated test suite.** Test cases in `test/` and `examples/` are manual input/output file pairs.
- To run: execute the compiled binary, which reads a `.dat` input file. It produces:
  - `.dat.ech` — echo of inputs
  - `.dat.out` — results
  - `.dat.ini` — initial conditions (for dynamic analyses)
- Test subdirectories: `simple/`, `dynamics/`, `strip/`, `vr7/`, `naca0015/`, `issues/`

## Documentation

- Sphinx docs in `doc/`. Build: `cd doc && make html`
- CI (`build_sphinx_doc.yml`) builds and deploys to gh-pages on push to `main`.
- PDF manuals are in `doc/` (not version-controlled output).
- `doc/requirements.txt` lists Sphinx dependencies (also duplicated in `pyproject.toml`).

## Python Environment

- Python 3.10, managed with `uv` (see `uv.lock`).
- Purpose: Sphinx documentation only — not used for the Fortran application.

---
> Source: [wenbinyugroup/gebt](https://github.com/wenbinyugroup/gebt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
