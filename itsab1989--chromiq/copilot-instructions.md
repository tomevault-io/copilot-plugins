## chromiq

> python3 -m venv .venv

# CLAUDE.md — ChromIQ

## Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install -r requirements-dev.txt   # test-only tools (pytest, pytest-qt)
```

## Run

```bash
source .venv/bin/activate
python main.py
```

## Test

```bash
source .venv/bin/activate
QT_QPA_PLATFORM=offscreen pytest      # ~20s, 724 tests
```

`pytest.ini` scopes collection to `tests/` (via `testpaths`). Without it a bare
`pytest` recurses into `.venv/` and — with `pytest-qt` active — collection
appears to hang for many minutes. If a run takes more than a few seconds,
something is wrong (a test opening a modal dialog `.exec()`, or `.venv` being
scanned again), not just "slow tests".

## Build distributable

```bash
source .venv/bin/activate
pip install pyinstaller
pyinstaller ChromIQ.spec
# Result: dist/ChromIQ.app
```

## Architecture

ChromIQ is a PyQt6 GUI for RGB printer ICC profiling with ArgyllCMS 3.5.0.

**Workflow**: `targen → printtarg → [print] → chartread → colprof`

### Module map

| Directory | Purpose |
|-----------|---------|
| `core/` | Settings, ArgyllRunner (QProcess), FileManager, logging, resource_path |
| `data/` | Patch capacity database, parameters.yaml (all CLI flags + tooltips) |
| `ui/` | All Qt widgets — main window, 4 tabs, shared TIFF preview, settings dialog |
| `workflow/` | Business logic — chart creation, PS generation, CUPS printing, measure, profile |

### Data flow

`parameters.yaml` → `ParameterWidget` rows in panels → `workflow/*.py` builds CLI args
→ `ArgyllRunner.run()` → `QProcess` → line_received signal → LogWidget + stripe detection

### Working-folder layout (Project / Run)

Every project is a folder under `~/ChromIQ/<target-name>/` owned by the
`Project` / `Run` / `Calibration` classes in `core/file_manager.py`:

```
<target-name>/             # = sanitised project name (spaces → hyphens)
  project.json             # manifest: schema_version, current_run, runs[]
  cal/                     # optional, shared across runs
    <target-name>-cal.*    # cal.ti1/.ti2/.ti3/.cal/.icc/_NN.tif
  exports/                 # i1Profiler exports (<target-name>-i1profiler.txt/.pxf)
  runs/run1/, run2/, …     # one folder per profile build
    <target-name>.*        # chart.ti1/.ti2/.cht/.ps/.channels.json + _NN.tif
    <target-name>.ti3      # the measurement (chartread output; averaged result reuses this stem)
    <target-name>.icc      # the profile (colprof output)
    reads/readN.ti3        # role-named, only when averaging is used
    preconditioning.ti3/.icc   # role-named, seeded by Project.new_run when refining
    merged.ti3/.icc        # role-named, build-time refinement-merge outputs
    calibrated.icc         # role-named, applycal output
    meta.json
```

**The chart's own files carry the sanitised project name as their stem**
(so printtarg stamps it on the printed sheet, the ICC is self-identifying,
Finder shows it). Derived/intermediate files (`reads/readN.ti3`,
`preconditioning.*`, `merged.*`, `calibrated.icc`) stay role-named — they
never go on paper. **The per-run folder still removes the need for any
prefix/suffix state encoding**, so the `pre_`/`cal_` prefixes and the
`_readN`/`_average`/`_merged` suffixes are gone. File stems follow
ArgyllCMS's natural coupling (`<name>.ti2` → `<name>.ti3` → `<name>.icc`),
so no post-tool renames are needed.

**All path construction goes through `Project` / `Run` / `Calibration`.**
Adding a new artefact = add a property/method to `Run`, never a stem pattern
elsewhere. `FileManager.project()` returns the current target's project;
`Run.for_dir(dir)` gives a project-less Run for path ops on a known folder.
Cross-run isolation makes the old "averaging reads double-counted into a
refinement merge" bug impossible by construction. See
`docs/dev_folder_layout.md`.

### Key patterns

**ArgyllRunner** is a singleton `QObject` injected into all workflow classes.
Only one process runs at a time — `is_running` guard checked before each operation.

**resource_path()** in `core/resource_path.py` resolves asset paths for both
development and PyInstaller frozen bundles.

**parameters.yaml** drives `ParameterWidget` creation automatically — add a new
parameter there and it appears in the UI without code changes.

**Patch capacity DB** in `data/patch_db.py` — empirical values from Argyll 3.1/3.5
measured with `printtarg -i<instr> -p<paper> -t300 -L`.  Unknown combos fall back
to binary search in `workflow/chart_creator.py`.

### Printing pipeline

TIFF → `PostScriptGenerator` (hex RGB, PS Level 2, exact PageSize, no scaling)
→ tempfile → `lp -d <printer> -o raw` — bypasses ColorSync and CUPS filters.

### ArgyllCMS binaries

Default path: `/Applications/Argyll/bin`  
Configurable in Settings dialog. The app shows a statusbar warning if binaries are missing.

### Translations (i18n)

UI language is a Settings option (restart-to-apply). `core/i18n.py` provides
`tr("English text")` — a plain string-catalog lookup against
`data/i18n/<code>.json` (key = exact English source string, miss = English).
`data/parameters.yaml` is translated separately by a
`data/i18n/parameters.<code>.yaml` overlay merged in `translate_parameters()`.

Rules when touching UI code:
- Wrap every new user-facing literal in `tr()`; runtime values use
  `tr("… {name} …").format(name=…)` (placeholders are part of the key).
- Count-bearing messages get explicit singular/plural variants, never `(s)`.
- After string changes run `python scripts/i18n_extract.py --missing de`
  and add the German translations — `tests/test_i18n.py` fails CI on missing
  keys, stale keys, placeholder mismatches, and over-long short labels.
- Adding a language = new `data/i18n/<code>.json` (with `@language_name`)
  + `parameters.<code>.yaml`; the Settings combobox discovers it automatically.
  `python scripts/i18n_agent/new_language.py <code> "<English>" "<Native>"`
  prints a complete, self-validating translation-agent prompt (style contract,
  hard rules, validation commands). Partial work staged in
  `data/i18n/staging/<code>.partial.json` is picked up automatically.

### Adding a parameter

1. Add entry to `data/parameters.yaml` with `tool`, `flag`, `type`, `default`, `tooltip_title`, `tooltip_body`.
2. Set `no_space: true` if value must be appended directly to flag (e.g. `-il` not `-i l`).
3. Set `expert_only: true` to hide in collapsed "Expert" section.
4. No code changes needed — `ParameterWidget` picks it up automatically.

### Adding a built-in (non-deletable) Create Chart preset

The Create Chart → Manual **Presets** dropdown can host hard-coded presets that
the user can't delete (e.g. "TC9.18 by Pharmacist", which loads a bundled `.ti1`
and runs printtarg only). The full mechanism, file/function map, gotchas, and a
step-by-step recipe are in **`docs/dev_builtin_presets.md`** — read it before
adding another.

---
> Source: [itsab1989/ChromIQ](https://github.com/itsab1989/ChromIQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
