## airzoo

> This file is for **any AI coding assistant** (Cursor, Claude Code, Codex, Windsurf, Copilot, etc.).

# Agent instructions: install AirZoo rendering client

This file is for **any AI coding assistant** (Cursor, Claude Code, Codex, Windsurf, Copilot, etc.).

When the user asks to install, set up, or configure AirZoo, **read this file and run the commands** in their environment. Do not only list steps — execute them when shell access is available.

**Before Step 1:** Tell the user the environment requirements below (same as README **Agent Quick Start**). Confirm they are on **Windows 11** with a supported GPU before proceeding. If they have RTX 50-series, warn about possible `simGetImages` issues.

## Requirements

- **OS:** Windows 11
- **GPU:** NVIDIA RTX 30- or 40-series recommended (RTX 50-series may have `simGetImages` read issues)
- **Tools:** Git, Conda (Anaconda/Miniconda)
- **Scene:** Switzerland only for the current public release (Cesium coverage)

## Installation workflow

Track progress:

```
Install Progress:
- [ ] Step 1: Clone repository
- [ ] Step 2: Create conda env airzoo
- [ ] Step 3: Copy AirSim settings.json
- [ ] Step 4: Download UE engine (Hugging Face)
- [ ] Step 5: Obtain airzoo-render.exe
- [ ] Step 6: (Optional) Test render
```

### Step 1: Clone

```bash
git clone https://github.com/nudt-sawlab/AirZoo.git
cd AirZoo
```

If the user already has the repo, `cd` into it and skip clone.

### Step 2: Conda environment

```bash
conda env create -f enviroment.yaml
conda activate airzoo
```

Verify: `python --version` should report 3.8.x.

### Step 3: AirSim camera intrinsics

Copy bundled settings to the user Documents folder (PowerShell):

```powershell
mkdir "$env:USERPROFILE\Documents\AirSim" -Force
copy .\config\settings.json "$env:USERPROFILE\Documents\AirSim\settings.json"
```

Defaults: **1920×1080**, **45° horizontal FOV** (`FOV_Degrees` is width-direction FOV under `CameraDefaults` in `config/settings.json`).

### Step 4: Obtain UE rendering engine

```powershell
pip install -U huggingface_hub
hf download choyaa/AirZoo-Engine `
  --repo-type dataset `
  --local-dir AirZooEngine
```

Source: https://huggingface.co/datasets/choyaa/AirZoo-Engine

> Prefer Cesium for Unreal with AirSim. Engine pack: https://huggingface.co/datasets/choyaa/AirZoo-Engine
> Prepared training renders are gated. If needed, request access at https://huggingface.co/datasets/choyaa/AirZoo-Data (do not treat this as an open download).

Launch: run your scene executable (e.g. `AirZooEngine/ObjectDetectionScene.exe` if provided), press **`F`** to connect AirSim, keep the window open during rendering.

### Step 5: airzoo-render.exe

Place `airzoo-render.exe` in the repo root (same folder as `airzoo_render_api.py`). It is **not** in git (see `.gitignore`). Obtain it from project releases or Hugging Face if distributed there.

### Step 6: Test render (optional)

With UE scene running:

```powershell
python .\airzoo_render_api.py `
  ".\examples\switzerland_seq1.txt" `
  ".\outputs\renders" `
  --renderer-exe ".\airzoo-render.exe" `
  --camera bottom_center `
  --map-load-delay 5 `
  --render-interval-delay 1 `
  --with-depth
```

Pose format (one line per frame, space-separated):

```
image_filename longitude latitude altitude roll pitch yaw
```

Recommend picking Switzerland coordinates that fall within Cesium coverage before writing the pose file.

## Verify geometry (optional)

After rendering two+ frames with depth, edit paths in `projection.py` and run:

```powershell
python projection.py
```

Output: `outputs/reprojection_matches.png`.

## Troubleshooting

| Issue | Action |
|-------|--------|
| `simGetImages` fails on RTX 50 | Use RTX 30/40 if available |
| Render outside Switzerland | Coordinates must be within Switzerland Cesium coverage |
| Missing `airzoo-render.exe` | Download separately. Pass `--renderer-exe` with full path |
| AirSim not connected | UE scene must be running. Press `F` in the scene window |
| Wrong image size / FOV | Re-copy or edit `%USERPROFILE%\Documents\AirSim\settings.json` |

## Notes for agents

- Run commands from the repo root unless the user specifies another path.
- On Windows, use PowerShell for paths with `$env:USERPROFILE`.
- Do not commit `airzoo-render.exe` or large render outputs.
- For full usage and CLI flags, see [README.md](README.md).

## Example user prompts

```
Read AGENTS.md and install AirZoo on my machine.
Set up the AirZoo rendering client following AGENTS.md.
Help me configure AirZoo — follow AGENTS.md step by step.
```

For usage questions and troubleshooting, see [AGENTS_FAQ.md](AGENTS_FAQ.md).

---
> Source: [nudt-sawlab/AirZoo](https://github.com/nudt-sawlab/AirZoo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
