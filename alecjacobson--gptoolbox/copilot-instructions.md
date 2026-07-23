## gptoolbox

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

gptoolbox is a MATLAB geometry processing toolbox. Nearly all code is plain `.m` files that run without compilation. The exception is `mex/`, which contains C++ mex functions that must be compiled separately.

## Adding to MATLAB path

```matlab
addpath(strjoin(strcat('/Users/ajx/Repos/gptoolbox/', ...
  {'external','imageprocessing','images','matrix','mesh','mex','quat','utility','wrappers'}),':'))
```

## Compiling mex files

```bash
cd mex
mkdir build && cd build
cmake ..
make
```

Mex files depend on Eigen, libigl, and optionally CGAL/Embree. If cmake can't find MATLAB, pass `-DMatlab_ROOT_DIR=<path>`. If cmake reports an unknown MATLAB version, update the `MATLAB_VERSIONS_MAPPING` variable in `mex/cmake/FindMATLAB.cmake`.

## Directory structure

- `mesh/` — triangle/tet mesh algorithms (the largest module): I/O, operators, deformation, parameterization, distances, queries
- `matrix/` — sparse matrix utilities, optimization solvers (quadprog wrappers, ADMM, active set)
- `utility/` — general MATLAB utilities (caching, figure export, path helpers)
- `wrappers/` — thin wrappers for external binaries: TetGen, Triangle, QSlim, meshfix, medit; `path_to_*.m` files tell gptoolbox where each binary lives
- `mex/` — compiled C++ mex functions (blank `.m` stubs provide `help` docs); uses libigl for geometry ops
- `external/` — vendored third-party MATLAB code (fast marching, colorbrewer, etc.)
- `imageprocessing/` — image colorization, diffusion curves, dithering
- `quat/` — quaternion arithmetic

## Coding conventions (from style.md)

**File naming:** `lowercase_with_underscores.m`. Exception: I/O functions use `readEXT` / `writeEXT` (e.g., `readOBJ`, `writePLY`). If the extension is ambiguous, use `readEXT_program`.

**Function header format** (required on every function):

```matlab
function [out1,out2] = my_function(V,F,varargin)
  % MY_FUNCTION Short description of what this does
  %
  % out1 = my_function(V,F)
  % [out1,out2] = my_function(V,F,'ParameterName',ParameterValue,...)
  %
  % Inputs:
  %   V  #V by 3 list of mesh vertex positions
  %   F  #F by 3 list of triangle mesh indices
  %   Optional:
  %     'ParamName' followed by description {default}
  % Outputs:
  %   out1  description
  %   out2  description
  %
  % See also: related_function
```

**Optional parameters** are handled via `varargin` + `containers.Map` pattern (see `centroid.m` or `arap.m` for the canonical idiom).

**Matrix size comments:** use `#V by dim` notation to document array dimensions in comments.

## Mex stubs

Each mex function has a `mex/funcname.m` that contains **only** the `%`-comment header block (no `function` line, no code). This makes `help funcname` show useful documentation in the MATLAB IDE even before or without compilation. The actual implementation lives in `funcname.cpp`.

---
> Source: [alecjacobson/gptoolbox](https://github.com/alecjacobson/gptoolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
