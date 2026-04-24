## usd-scene-physics-prep

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

A USD scene toolset for **asset splitting** and **physics simulation preprocessing**: normalize raw scenes into a reusable model/material library with scene reference structures, then batch-assign collision bodies, rigid bodies, and joints for Omniverse Isaac Sim interaction and navigation simulation.

## Environment

- **Core dependencies**: `usd-core` (pxr), `numpy`, `pandas`
- **Physics preprocessing** (interaction/navigation): requires Omniverse Isaac Sim (`isaacsim`, `omni.*`)
- **Doc management**: `pip install pyyaml`

Isaac Sim scripts must be run via the Isaac Sim Python wrapper:
```bash
# Auto-detects Isaac Sim install; or set ISAAC_SIM_ROOT=/path/to/isaac_sim-<version>
./scripts/isaac_python.sh <script.py> [args...]
```

## Common Commands

**Step 1 – Scene splitting** (standard Python, no Isaac Sim):
```bash
python clean_data.py
# Reads home_scenes/<scene_id>/{start_result_fix.usd|start_result_new.usd}
# Writes target/ (Materials, models, scenes)
```

**Step 2 – Interaction physics** (requires Isaac Sim):
```bash
# Edit scene_path/dest_scene_path in the script first
./scripts/isaac_python.sh set_physics/preprocess_for_interaction.py
# Output: start_result_dynamic.usd
```

**Step 2 alt – Navigation physics** (requires Isaac Sim):
```bash
# Edit paths first
./scripts/isaac_python.sh set_physics/preprocess_for_navigation.py
# Output: start_result_navigation.usd
```

**Step 3 – Export/package**:
```bash
./scripts/isaac_python.sh set_physics/get_all_references.py
./scripts/isaac_python.sh set_physics/export_scene.py
```

**One-shot SimReady CLI** (Option B – no manual splitting needed):
```bash
./scripts/isaac_python.sh -m set_physics.simready --input-usd /path/to/scene.usd
```

**External /root-structured scenes** (Option C – for SimBench/GRScene datasets):
```bash
./scripts/isaac_python.sh scripts/prep_interaction_root_scene.py \
  --input /path/to/scene.usd \
  --output /path/to/scene_interaction_dynamic.usd
```

**Normalized export** (specs_normalizer):
```bash
# GRScenes
python -m specs_normalizer --src-target ./target --dst-root ./export_specs \
  --asset-name GRScenes_assets --scene-name GRScenes100 --scene-category home --with-annotations
# MesaTask
python -m specs_normalizer --src-target ./target --dst-root ./export_mesa_specs \
  --asset-name MesaTask_assets --scene-name MesaTask --scene-category office_table
```

**Fix MDL/texture absolute paths in normalized assets**:
```bash
python scripts/fix_normalized_mdl_paths.py \
  --dataset-root /path/to/GRScenes-test1-normalized \
  [--materials-dir /path/to/GRScenes-test1-normalized/Material/mdl] \
  [--category bottle] [--dry-run] [--workers 8]
```

**Fix portable texture paths**:
```bash
./scripts/isaac_python.sh scripts/collect_textures_to_local_textures.py \
  --input /abs/path/to/scene.usd \
  --output /abs/path/to/scene_textures_fixed.usd \
  --report /abs/path/to/texture_rewrite_report.json
```

**Shape-invariant dedup** (complementary to geom_only):
```bash
python scripts/report_asset_mesh_dedup.py \
  --dataset-root ./GRScenes-test1-normalized \
  --dataset-name GRScenes_assets \
  --category bottle \
  --mode shape_invariant \
  --hausdorff-threshold 0.05 \
  --merge-tolerance 0.005 \
  --out-dir check_reports/shape_invariant/
```

**Documentation maintenance**:
```bash
python scripts/doc_manager.py --find-refs simready.py   # find docs referencing a code file
python scripts/doc_manager.py --validate                 # validate doc metadata
python scripts/doc_manager.py --gen-index               # regenerate docs/INDEX.md
```

## DLC Remote Job Submission

Scripts in `scripts/dlc/` submit physics preprocessing jobs to Alibaba Cloud PAI-DLC GPU workers using the Isaac Sim 4.5.0 container image; inside the container the `dlc-operator` Claude agent can be used for job monitoring and submission automation.

**Single job**:
```bash
bash scripts/dlc/launch_job.sh <name> <chunk_id> <chunk_total> [data_sources] [command_args]
```

**Batch submission** (one job per chunk):
```bash
python scripts/dlc/submit_batch.py --name <name> --total <N> --mode <mode> [--command_args "..."]
```

**Execution modes** (first arg to `run_task.sh`):
- `interaction` – interaction physics preprocessing (Isaac Sim)
- `navigation` – navigation physics preprocessing (Isaac Sim)
- `simready` – one-shot SimReady CLI (Isaac Sim)
- `prep_root_scene` – external `/root`-structured scene prep (Isaac Sim)
- `normalize` – normalized export via `specs_normalizer` (Isaac Sim Python)
- `normalize_assets` – asset transform normalization (recenter + Y-up→Z-up) via `normalize_asset_transforms.py` (Isaac Sim Python)
- `dedup` – asset dedup report by category chunk via `scripts/dlc/dedup_by_category.py` (Isaac Sim Python)
- `clean` – scene splitting via `clean_data.py` (Isaac Sim Python)
- `custom` – pass any script directly to Isaac Sim Python
- `batch` – chunk-based dispatch (default; passes extra args to Isaac Sim Python)

**Environment**: uses Isaac Sim 4.5.0 container image; activated via `scripts/isaac_python.sh` (not conda). Override defaults via `DLC_WORKSPACE_ID`, `DLC_RESOURCE_ID`, `DLC_IMAGE`, `DLC_CODE_ROOT`, `DLC_BIN`. The `dlc` binary is gitignored and must be placed at the project root.

## Architecture

### Three SimReady Options

| Option | Entry Point | Use Case |
|--------|-------------|----------|
| A – Standard Pipeline | `set_physics/preprocess_for_interaction.py` | Batch processing with fine control (split first, then bind physics) |
| B – One-shot CLI | `set_physics/simready.py` | Quick start; no manual intermediate steps |
| C – External Adapter | `scripts/prep_interaction_root_scene.py` | Non-standard `/root`-structured datasets (SimBench/GRScene) |

### Core Pipeline (Option A)

1. **Scene splitting** (`clean_data.py` → `set_physics/pxr_utils/data_clean.py:parse_scene`)
   - Copies root structure, skips `Meshes/Looks/PhysicsMaterial`
   - Extracts each instance into `instance.usd`, replaces with reference
   - Stable model fingerprint via MD5 hash of reference chain + transform (`data_clean.py:612-616`)
   - Creates `target/scenes/<sid>/Materials` and `models` symlinks

2. **Transform normalization** (`data_clean.py:121-149`)
   - Converts `xformOp:transform` → `translate/orient/scale` for physics API compatibility

3. **Interaction preprocessing** (`set_physics/preprocess_for_interaction.py:335-387`)
   - Binds rigid body + collision approximation for pickable/articulated objects
   - Enables joints; sets up collision groups + filtering

4. **Navigation preprocessing** (`set_physics/preprocess_for_navigation.py:198-231,389-425`)
   - Disables non-primary doors; applies static triangle mesh approximation; writes semantic labels

5. **Export** (`get_all_references.py` + `export_scene.py`)
   - Scans model/material references → JSON manifest → copies files to destination

### Directory Layout

**Input**:
```
home_scenes/<scene_id>/{start_result_fix.usd|start_result_new.usd}
home_scenes/<scene_id>/Materials/*.mdl + Textures/
```

**Pipeline output** (`target/`):
```
target/
  Materials/Textures/  *.mdl
  models/
    layout|object/
      articulated|others/
        <category>/<model_hash>/instance.usd
  scenes/<scene_id>/
    Materials -> ../../Materials  (symlink)
    models    -> ../../models     (symlink)
    start_result_{new|fix}.usd
```

**Normalized export** (`export_specs/`):
```
export_specs/
  Material/mdl/{mid}.mdl + textures/
  GRScenes_assets/<category>/<model_hash>/usd/<model_hash>.usd
  GRScenes100/home/<scene_id>/layout.usd
```

### Key Modules

- `set_physics/pxr_utils/data_clean.py` – Core splitting logic (`parse_scene`, `recursive_copy`, `create_instance`, `unique_id`)
- `set_physics/pxr_utils/usd_physics.py` – Physics API helpers (collider binding, rigid body)
- `set_physics/pxr_utils/read_info.py` – CSV reading utilities
- `specs_normalizer/` – Normalized export package: `normalize.py` (orchestrator) → `validators/structure.py`, `exporters/{materials,assets,scenes}.py`
- `scripts/doc_manager.py` – Doc metadata validation and index generation
- `set_physics/tools/` – Isaac Sim utilities (thumbnail generation, random model placement)
- `scripts/` – One-off and batch automation scripts (many `oneoff_*.py` for targeted fixes)
- `scripts/dlc/` – DLC remote job submission: `launch_job.sh` (single job), `submit_batch.py` (batch), `run_task.sh` (in-container dispatcher)

### Physics Collision Approximation Constants

Used across multiple scripts; named constants for Isaac Sim PhysX schemas:
- `"sdf"` – SDF
- `"convexHull"` – Convex Hull
- `"convexDecomposition"` – Convex Decomposition
- `"meshSimplification"` – Mesh Simplification
- `"none"` – Triangle Mesh (static)

## Documentation Maintenance

All docs under `docs/` use YAML frontmatter (`title`, `code_reference`, `created_at`, `updated_at`, `maintainer`, `status`). When modifying code, run `python scripts/doc_manager.py --find-refs <file>` to find related docs to update. After modifying docs, run `--validate` and `--gen-index`.

## Agent Team Documentation Rule

**When using `TeamCreate` / `Agent` tool to spawn teammate agents, you (the team lead) MUST inject the documentation requirement into each agent's `prompt` parameter.** Spawned subagents do NOT automatically read CLAUDE.md, so the rule must be explicitly passed to them.

### What to inject

Append the following block to every spawned agent's prompt (adapt paths as needed):

```
【Documentation Requirement】
You MUST document your work before finishing. This is mandatory.
- What to document: research findings, code changes, test commands & results, decisions, errors & resolutions.
- Where to write:
  • Results/progress → `docs/` (with YAML frontmatter: title, code_reference, created_at, updated_at, maintainer, status)
  • Task logs → project memory at `~/.claude/projects/-cpfs-shared-simulation-zhuzihou-dev-usd-scene-physics-prep/memory/`
- If you have write permission: write docs directly.
- If you are read-only (e.g., Explore agent): send all findings via SendMessage to the team lead, including enough detail to produce the doc.
- Timing: document as you go, not just at the end. Each major milestone should be recorded.
```

### Checklist for team lead

- [ ] When calling `Agent` with `team_name`, include the documentation block above in the `prompt`.
- [ ] When using `SendMessage` to assign new tasks to existing teammates, remind them of the documentation requirement if the task scope is significant.
- [ ] After all teammates finish, verify that documentation was produced (check `docs/` and memory files).

## Important Notes

- **Windows**: symlink creation and `cp` commands may not work; use `shutil` alternatives. See `docs/operations/windows_notes.md`.
- **Texture portability**: USD files may embed machine-specific `/tmp/<hash>/textures/` paths. Use `collect_textures_to_local_textures.py` to fix before distribution.
- **Texture symlinks in exports**: `textures -> ../../../Material/mdl/textures` symlinks inside each asset's `usd/` directory are NOT created by `specs_normalizer` — they are created by a separate post-processing script.
- **`/root` structure**: For external datasets (SimBench/GRScene task10) that use `/root` instead of `/Root/Meshes`, use `scripts/prep_interaction_root_scene.py` (Option C), not the standard pipeline.
- **MDL copy fix**: If `.mdl` files are missing in the export, they need to be explicitly copied (see commit notes on `test1` MDL copy+placeholder fix).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jandan138) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
