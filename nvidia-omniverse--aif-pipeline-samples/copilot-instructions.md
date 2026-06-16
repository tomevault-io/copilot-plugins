## aif-pipeline-cli

> AIF Pipeline CLI command reference. Maps natural language requests to correct CLI commands.


# AIF Pipeline CLI Reference

The `aif-pipeline` CLI provides a unified interface for CAD-to-USD asset processing. All commands require `uv run` prefix or an activated virtual environment.

## Command Lookup

### Setup & Configuration

| User Intent | Command |
|---|---|
| Set up / configure Kit path | `aif-pipeline config add <name> --from <kit-project-root>` |
| Switch to a different Kit version | `aif-pipeline config use <name>` |
| Show current configuration | `aif-pipeline config show` |
| List all configurations | `aif-pipeline config list` |
| Edit configuration | `aif-pipeline config edit` |
| Set a specific config value | `aif-pipeline config set <key> <value>` |
| Create project-specific config | `aif-pipeline config add --project --from <kit-root>` |
| Use a config for one command only | `aif-pipeline --config <name> <command> ...` |

The `--from` flag takes the **Kit project root** (the directory containing `repo.bat`/`repo.sh` or the NGC extract root). Do not pass `_build/` or subdirectories.

### CAD Conversion

**Agent behavior:** When a user asks to convert CAD files:

1. Determine CAD format to select the correct spec file (see table below).
2. Check Kit config (`aif-pipeline config show`) - conversion requires Kit.
3. If no Kit config exists, guide setup with `aif-pipeline config add` first.
4. Confirm input directory, output directory, and spec before running.

| User Intent | Command |
|---|---|
| Convert CAD files to USD | `aif-pipeline convert INPUT OUTPUT --spec <spec.json>` |
| Convert with parallelism | `aif-pipeline convert INPUT OUTPUT --spec <spec.json> --concurrent 64` |
| Resume interrupted conversion | `aif-pipeline convert INPUT OUTPUT --spec <spec.json> --skip-existing` |

**Spec file by CAD format:**

| Format | Extensions | Spec File |
|---|---|---|
| Creo/PTC | `.prt`, `.asm` | `scripts/data/creo_spec.json` |
| JT | `.jt` | `scripts/data/jt_spec.json` |
| DGN | `.dgn` | `scripts/data/dgn_spec.json` |

### Scene Optimization

**Agent behavior:** When a user asks to optimize assets:

1. Ask if they have a vendor-specific preset or want the generic one (`so/generic/generic_preset.json`).
2. Ask if they want non-destructive (separate output directory) or in-place.
3. Check Kit config (`aif-pipeline config show`) - optimization requires Kit.
4. For custom presets, read `.cursor/rules/scene-optimizer-presets.mdc` for the operation catalog and ordering rules.

| User Intent | Command |
|---|---|
| Optimize assets (non-destructive) | `aif-pipeline optimize INPUT OUTPUT --preset <preset.json>` |
| Optimize assets in-place | `aif-pipeline optimize INPUT --preset <preset.json>` |
| Optimize with specific preset | `aif-pipeline optimize INPUT OUTPUT --preset so/generic/generic_preset.json` |
| Optimize with dynamic preset | `aif-pipeline optimize INPUT OUTPUT --preset <preset.json> --dynamic-preset` |

Preset paths are relative to the repo root. Common presets:
- `so/generic/generic_preset.json` — general-purpose CAD asset optimization
- `so/spt/gb300/gb300.json` — GB300 rack optimization
- Vendor presets in `so/vertiv/<model>/`, `so/trane/`

**Dynamic preset** (`--dynamic-preset`): Injects native mesh paths per file, letting Scene Optimizer tailor operations to each asset's specific geometry. Useful when the generic preset is too aggressive or too conservative for mixed-complexity assets.

### Validation

**Agent behavior:** When a user asks to "validate" assets:

1. Check for an active Kit config (`aif-pipeline config show`). If Kit is already configured, offer both paths directly.
2. Ask whether they want **Kit-based** or **OAV standalone** validation.
3. If no Kit config exists, note that Kit validation needs setup first (`aif-pipeline config add`) and suggest OAV as the ready-to-use option.
4. Present the available options for the chosen path and let them choose before running any command.

- **Kit validation** — requires Kit and GPU; runs built-in + Scene Optimizer C-accelerated rules
- **OAV standalone** — no Kit/GPU; runs custom AIF rules from `oav/aif_validators/`

| User Intent | Command |
|---|---|
| Validate assets (Kit-based) | `aif-pipeline validate INPUT OUTPUT` |
| Pre-metadata validation | `aif-pipeline validate INPUT OUTPUT --stage pre` |
| Post-metadata validation | `aif-pipeline validate INPUT OUTPUT --stage post` |
| Validate and auto-fix | `aif-pipeline validate INPUT OUTPUT --fix --output-assets FIXED_DIR` |
| Validate against a feature | `aif-pipeline validate INPUT OUTPUT --feature minimal_placeable_visual` |
| Per-rule timing diagnostics | `aif-pipeline validate INPUT OUTPUT --fine-grained` |

OAV standalone validation (no Kit/GPU required):
```bash
uv run --directory oav validate --category AIF /path/to/asset.usd
uv run --directory oav validate --rule AIFMetadataChecker /path/to/asset.usd
```

### Metadata

| User Intent | Command |
|---|---|
| Create metadata template | `aif-pipeline metadata create --type <type> --output <file>` |
| Apply metadata to USD | `aif-pipeline metadata apply INPUT --output OUTPUT --prim <name>` |
| Convert electrical CSV to USDC | `aif-pipeline metadata electrical INPUT OUTPUT --prim <name>` |

Equipment types: `cdu` (81 props), `crah` (51 props), `ups` (51 props), `gb300_rack` (28 props)

**Agent behavior:** When a user asks about metadata, walk them through the full equipment metadata workflow:

1. **Create** template: `aif-pipeline metadata create --type <type> --output metadata.json`
2. **Edit** the JSON to fill in equipment-specific values
3. **Apply** to generate USDA: `aif-pipeline metadata apply metadata.json --output <Model>_Properties.usda --prim <DefaultPrim>`
4. **Compose** as sublayer into the main asset (use `add_layers.py` processor or manual sublayering)
5. **Validate**: `uv run --directory oav validate --rule AIFMetadataChecker <asset>`
6. **Update** when templates change: `aif-pipeline metadata update <usda_file> [--type <type>] [--dry-run]`

For property details and naming conventions, read `.cursor/rules/usd-aif-profile.mdc`.

### Full Pipeline

| User Intent | Command |
|---|---|
| Run complete pipeline | `aif-pipeline run INPUT OUTPUT --spec <spec> --preset <preset>` |
| Skip validation step | `aif-pipeline run INPUT OUTPUT --no-validate --spec <spec> --preset <preset>` |
| Run specific steps only | `aif-pipeline run INPUT OUTPUT --steps optimize,validate --preset <preset>` |

Pipeline order: convert → optimize → validate (OAV/AIF rules on optimized assets)

## Configuration Priority

Highest to lowest:
1. **Project config** — `./aif-pipeline.yaml` in current directory
2. **Active user config** — `~/.aif-pipeline/config.yaml`
3. **Named config** — via `--config <name>`
4. **CLI flags** — `--preset`, `--concurrent`, etc.
5. **Defaults**

## Common Flags

| Flag | Description | Default |
|---|---|---|
| `--concurrent N` | Parallel processes | Varies (4 for optimize, 10 for validate, 64 for convert) |
| `--skip-existing` | Skip already-processed files | false |
| `--timeout SECONDS` | Per-file timeout | Varies by command |
| `--kit-path PATH` | Override Kit executable | From config |

## Getting Help

```bash
aif-pipeline --help           # List all commands
aif-pipeline <command> --help  # Help for specific command
```

---
> Source: [NVIDIA-Omniverse/aif-pipeline-samples](https://github.com/NVIDIA-Omniverse/aif-pipeline-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
