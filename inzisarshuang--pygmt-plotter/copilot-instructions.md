## pygmt-plotter

> This file defines the rules an AI coding agent or human contributor should read before changing this repository.

# PyGMT-Plotter Coding Baseline

This file defines the rules an AI coding agent or human contributor should read before changing this repository.

## Scope

- Keep plotting workflows configuration-driven. Scientific paths, units, scales, color limits, transparency, and styles belong in cfg files, not hidden constants in entry scripts.
- Reuse `lib/pygmt_io.py`, `lib/pygmt_geo.py`, and `lib/pygmt_visual.py` before adding helpers.
- Keep configuration and file mechanics in `pygmt_io`, spatial conversion and sampling in `pygmt_geo`, and figure composition in `pygmt_visual`.
- Add domain, mask, or export modules only when the repository has independent behavior to own; do not create empty pipeline stages.
- Add a shared function only when it removes real duplication or establishes one scientific behavior in one place.
- Preserve existing cfg keys unless a migration path and compatibility alias are provided.

## Scientific Data Rules

- Never silently change units, sign conventions, nodata meaning, CRS, transform, or registration.
- Validate raster dimensions, transform, and CRS before pixel-wise arithmetic.
- Treat `0` and nodata as different values. Convert one to the other only through an explicit option.
- Record scale factors and display ranges in cfg files. A visual color limit must not clip or alter source data.
- Resolve relative paths from the cfg file directory so the same command works from any current directory.

## Memory And Output Safety

- Read large rasters by block/window and large tables by chunks. Avoid full-matrix copies when a streaming path exists.
- Prefer passing files to GMT over materializing full grids as Pandas DataFrames.
- Use unique temporary files and remove them in `finally` or a context manager.
- Write outputs to a temporary sibling and atomically replace the destination after success.
- Cache derived DEM grids only with a source signature that includes path, size, and modification time.

## Python Style

- Target Python 3.10 or newer and use type hints for public functions.
- Use `pathlib.Path`, context managers, and `subprocess.run(..., check=True)`.
- Raise actionable exceptions; do not use `assert` for runtime validation.
- Start every new production Python file with the GAMMA-InSAR-style structured docstring containing the module name, `功能概述:`, and `函数说明:` sections. List every public top-level function and class, and update the list whenever an interface is added, removed, or renamed. The required template is in `docs/development/python_rules.md`.
- Start every production function and method, including private helpers and entry-script orchestration, with a concise docstring that states its purpose. Public scientific APIs may add bilingual parameter and return details when useful.
- Entry scripts should parse configuration, orchestrate shared functions, and remain thin.

## Configuration Files

- Use `key = value`, one setting per line.
- Reject malformed lines, duplicate keys, unknown keys, invalid choices, and inconsistent ranges early.
- Keep hexadecimal colors such as `#1f77b4` valid. Inline comments require whitespace before `#`.
- Document every supported key in the corresponding example cfg, including units and valid choices.

## Documentation

- Keep every README bilingual: place each English heading or passage first and its corresponding Chinese translation immediately after it.
- Use numbered bilingual sections in root and workflow READMEs, such as `## 1. Project Structure / 项目结构` and `### 1.1 Documentation / 文档入口`. Keep numbering continuous after structural edits.
- Organize the root README in this order when applicable: project summary, project structure and documentation entry points, workflow overview, dependencies and installation, quick start, data/output behavior, verification and development, authors, security, and license notice.
- Give each list item one clear subject. Put its English description first and the matching Chinese description after `<br>` in the same item; keep both languages semantically equivalent and in the same order.
- Keep command examples executable from the stated directory. Use `<placeholder>` for user-supplied values and place English comments before their Chinese counterparts.
- Use repository-relative links, verify every internal link, and never place credentials, private absolute paths, region-specific results, or generated products in a README.
- Update the relevant README whenever workflows, public modules, cfg interfaces, dependencies, output behavior, or verification commands change.

## Verification

Run before committing:

```bash
python -m py_compile lib/*.py plot_defo_dem_optic/*.py plot_defo_profile/*.py
python -m unittest discover -s tests -v
```

Run both example plotting workflows when shared plotting or I/O behavior changes. Inspect the generated images, not only process exit codes.

Do not add region-specific performance benchmarks. Datasets vary too much for a fixed benchmark to be meaningful here.

---
> Source: [inzisarshuang/PyGMT-Plotter](https://github.com/inzisarshuang/PyGMT-Plotter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
