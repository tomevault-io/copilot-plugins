## aif-pipeline-samples

> Sample scripts and presets for creating SimReady USD assets in NVIDIA Omniverse. Covers CAD ingestion, Scene Optimizer optimization, validation, and metadata workflows for AI Factory digital twin applications.

# AIF Pipeline Samples

Sample scripts and presets for creating SimReady USD assets in NVIDIA Omniverse. Covers CAD ingestion, Scene Optimizer optimization, validation, and metadata workflows for AI Factory digital twin applications.

## Repository Layout

| Directory | Purpose |
|-----------|---------|
| `cli/` | Unified `aif-pipeline` CLI tool (convert, optimize, validate, metadata, run, config) |
| `scripts/` | Pipeline scripts for batch processing; `scripts/examples/` has simpler demos |
| `so/` | Scene Optimizer presets — `generic/` template plus vendor examples (`vertiv/`, `spt/`, `trane/`) |
| `so/generic/lib/` | Reusable Python processor scripts for presets |
| `oav/` | Custom USD validation rules (standalone, no Kit/GPU required) |
| `metadata/` | Metadata templates and tools for AIF equipment properties |
| `connections/` | Connection points workflow (thermal, electrical, airflow interfaces) |
| `asset_processor/` | Browser-based visual preset builder for Scene Optimizer |
| `docs/` | Sphinx/MyST documentation source |
| `assets/` | Sample USD assets |

## Core Workflow

The standard pipeline is: **Convert** → **Optimize** → **Validate** → **Metadata**

```bash
# Full pipeline in one command
aif-pipeline run input_cad/ output/ --spec scripts/data/creo_spec.json --preset so/generic/generic_preset.json

# Or step by step
aif-pipeline convert input_cad/ output_usd/ --spec scripts/data/creo_spec.json
aif-pipeline optimize output_usd/ optimized/ --preset so/generic/generic_preset.json
aif-pipeline validate optimized/ validation/
aif-pipeline metadata apply metadata.json --output metadata.usda --prim Root
```

All commands require `uv run` prefix or an activated venv (`.venv/Scripts/Activate.ps1` on Windows, `source .venv/bin/activate` on Linux).

## Agent Knowledge Files

When working with USD assets or the pipeline, read these rule files for domain-specific guidance:

| File | When to Read |
|------|-------------|
| `.cursor/rules/aif-pipeline-cli.mdc` | **Always** — CLI command reference with natural language mappings |
| `.cursor/rules/usd-universal.mdc` | Validation fails, user reports visual artifacts, or you need to map a failed rule to a Scene Optimizer fix |
| `.cursor/rules/usd-aif-profile.mdc` | User mentions metadata, connection points, equipment types (CDU/CRAH/UPS/GB300), or `aif:` properties |
| `.cursor/rules/scene-optimizer-presets.mdc` | User wants to create, edit, customize, or understand a Scene Optimizer preset JSON |
| `.cursor/rules/usd-issues-catalog.mdc` | User reports a specific error, rendering problem, or you need to diagnose a known failure pattern |

## Conventions

- **License headers:** All source files use SPDX MIT headers (`SPDX-FileCopyrightText: Copyright (c) 2024-2026 NVIDIA CORPORATION & AFFILIATES`)
- **Python:** 3.10-3.12, managed with `uv`; snake_case naming
- **Package manager:** `uv` (not pip) — `uv sync` to install, `uv run` to execute
- **Documentation:** Markdown with MyST syntax (Sphinx-compatible); use regular hyphen dashes (` - `), not em/en dashes
- **Presets:** JSON arrays of operation objects in `so/` directory
- **Metadata namespaces:** `aif:core:` (common properties), `aif:spec:` (equipment-specific)

## Agent Behavior

### General Principles

- **Check environment first:** Run `aif-pipeline config show` before any Kit-dependent command (convert, optimize, Kit-based validate). If Kit is not configured and the task requires it, guide setup with `aif-pipeline config add <name> --from <kit-root>` before proceeding.
- **Prefer non-destructive operations:** When optimizing, default to a separate output directory unless the user explicitly asks for in-place.
- **`uv run` prefix:** All commands need `uv run` prefix unless the user has activated a venv.

### Conversion Requests

When a user asks to convert CAD files to USD:

1. **Determine CAD format** to select the correct spec file:
   - Creo/PTC (`.prt`, `.asm`): `scripts/data/creo_spec.json`
   - JT (`.jt`): `scripts/data/jt_spec.json`
   - DGN (`.dgn`): `scripts/data/dgn_spec.json`
2. **Check Kit config** - conversion requires Kit.
3. If no Kit config exists, guide setup first.
4. Confirm input directory, output directory, and spec before running.

### Optimization Requests

When a user asks to optimize USD assets:

1. **Ask about the preset:**
   - Generic: `so/generic/generic_preset.json` (good default for most CAD assets)
   - Vendor-specific: check `so/vertiv/`, `so/spt/`, `so/trane/` for existing presets matching their equipment
2. **Ask destructive vs. non-destructive:** separate output directory (default) or in-place.
3. **Check Kit config** - optimization requires Kit.
4. If they want to customize the preset, read `.cursor/rules/scene-optimizer-presets.mdc` for the operation catalog and ordering rules.

### Validation Requests

When a user asks to validate assets:

1. **Check for an active Kit config** by running `aif-pipeline config show`. If Kit is already configured, offer both validation paths directly.
2. **Ask which path** they want to use:
   - **Kit-based validation** (`aif-pipeline validate`) - requires Kit and GPU, runs built-in + Scene Optimizer C-accelerated rules
   - **OAV standalone validation** (`uv run --directory oav validate`) - no Kit/GPU required, runs custom AIF rules
3. If no Kit config exists, note that Kit validation requires setup first (`aif-pipeline config add`) and suggest OAV as the ready-to-use option.
4. Based on their choice, present the available options and let them choose before running:
   - **Kit options:** stage (pre/post), feature targeting, auto-fix, fine-grained timing
   - **OAV options:** full AIF category (`--category AIF`), or specific rule (`--rule <RuleName>`)

See `docs/reference/validators.md` for the full comparison.

### Metadata Requests

When a user asks about metadata or equipment properties:

1. **Determine equipment type:**
   - CDU (Coolant Distribution Unit) - 81 properties
   - CRAH (Computer Room Air Handler) - 51 properties
   - UPS (Uninterruptible Power Supply) - 51 properties
   - GB300 Rack - 28 properties (pre-filled with NVIDIA values)
2. **Walk through the full cycle:**
   a. Create template: `aif-pipeline metadata create --type <type> --output <file>.json`
   b. User fills in values (show them what properties exist for that type)
   c. Apply: `aif-pipeline metadata apply <file>.json --output <Model>_Properties.usda --prim <DefaultPrim>`
   d. Compose as sublayer into main asset
3. **Validate metadata:** `uv run --directory oav validate --rule AIFMetadataChecker <asset>`
4. Read `.cursor/rules/usd-aif-profile.mdc` for property details and naming conventions.

### Full Pipeline Requests

When a user asks to run the complete pipeline:

1. Confirm input directory (CAD files) and output directory.
2. Determine spec file (ask CAD format - see Conversion above).
3. Determine preset (ask or default to `so/generic/generic_preset.json`).
4. Check Kit config.
5. Show the command and confirm before running:
   `aif-pipeline run INPUT OUTPUT --spec <spec> --preset <preset>`
6. After completion, suggest validation review and metadata application.

### Validation Failure Fix Loop

When validation reports failures:

1. Read `.cursor/rules/usd-universal.mdc` for the symptom-to-fix mapping tables and quick rule-to-operation lookup.
2. Map each failed rule to the corresponding Scene Optimizer operation.
3. If multiple fixes are needed, compose them into a preset following the operation ordering in `.cursor/rules/scene-optimizer-presets.mdc`.
4. Run the fix preset, then re-validate.
5. For AIF-specific failures (`AIFMetadataChecker`, etc.), read `.cursor/rules/usd-aif-profile.mdc`.

### Troubleshooting

Common errors and recovery:

- **"Kit path not configured"** - Run `aif-pipeline config add <name> --from <kit-root>`
- **Timeout during conversion/optimization** - Increase with `--timeout SECONDS`; check asset complexity
- **Out of memory** - Reduce `--concurrent` value; each Kit instance uses 4-8 GB RAM
- **Partial batch failure** - Use `--skip-existing` to resume from where it left off
- **"Command not found"** - Prefix with `uv run` or activate venv first
- **OAV ImportError** - Run `uv sync` from repo root to install dependencies

For more error patterns, see `.cursor/rules/usd-issues-catalog.mdc`.

## Key Reference Files

- `cli/README.md` — Full CLI command documentation
- `so/generic/generic_preset.json` — Canonical Scene Optimizer preset
- `so_operations.json` — Complete Scene Optimizer operation parameter reference
- `docs/reference/scene-optimizer-presets.md` — Preset authoring guide
- `metadata/templates/common_properties.py` — AIF common metadata properties
- `docs/reference/validators.md` — Validation rules reference

## Cross-Reference Guide

- **Validation rule failed?** Find the rule name in `usd-universal.mdc` symptom-to-fix tables, then compose the fix using `scene-optimizer-presets.mdc`
- **Need equipment metadata?** Start with `usd-aif-profile.mdc` for property types and naming, use CLI commands from `aif-pipeline-cli.mdc`
- **Known bug or rendering artifact?** Check `usd-issues-catalog.mdc` before investigating from scratch
- **Need a library script?** Documented in `usd-universal.mdc` (when to use) and `scene-optimizer-presets.mdc` (how to call from presets)

---
> Source: [NVIDIA-Omniverse/aif-pipeline-samples](https://github.com/NVIDIA-Omniverse/aif-pipeline-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
