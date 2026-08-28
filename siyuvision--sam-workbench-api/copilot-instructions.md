## sam-workbench-api

> Instructions for coding agents installing or changing **sam-workbench-api**. Humans: see [README.md](README.md).

# AGENTS.md

Instructions for coding agents installing or changing **sam-workbench-api**. Humans: see [README.md](README.md).

## What this repo is

Local FastAPI SAM server for the QuPath plugin:

https://github.com/siyuvision/qupath-sam-workbench

This process must be running **before** QuPath is used. Bind `127.0.0.1:8000`, **one** Uvicorn worker. Do not expose it on a LAN.

Paired versions: API **0.1.3**, plugin **0.1.6**.

## Do this (install for a user)

1. Python **3.12** venv. Install a matching PyTorch build from https://pytorch.org/get-started/locally/ (CUDA if NVIDIA GPU, else CPU). Then:

```bat
python -m pip install .
```

On Windows, SAM2/SAM3 may need `triton-windows`. Pins are in `pyproject.toml` (git deps for MobileSAM / sam2 / sam3).

2. Checkpoints are **local files only**. The API will **not** download from Hugging Face. Missing files → HTTP 404 with the expected path.

   Put files in `%USERPROFILE%\.samapi\<type>\<filename>`:

   | type | file | URL |
   | --- | --- | --- |
   | `sam2_s` (try this first) | `sam2_hiera_small.pt` | https://dl.fbaipublicfiles.com/segment_anything_2/072824/sam2_hiera_small.pt |
   | `sam2_t` | `sam2_hiera_tiny.pt` | https://dl.fbaipublicfiles.com/segment_anything_2/072824/sam2_hiera_tiny.pt |
   | `sam2_bp` | `sam2_hiera_base_plus.pt` | https://dl.fbaipublicfiles.com/segment_anything_2/072824/sam2_hiera_base_plus.pt |
   | `sam2_l` | `sam2_hiera_large.pt` | https://dl.fbaipublicfiles.com/segment_anything_2/072824/sam2_hiera_large.pt |
   | `vit_t` | `mobile_sam.pt` | https://github.com/ChaoningZhang/MobileSAM/raw/master/weights/mobile_sam.pt |
   | `vit_b` | `sam_vit_b_01ec64.pth` | https://dl.fbaipublicfiles.com/segment_anything/sam_vit_b_01ec64.pth |
   | `vit_l` | `sam_vit_l_0b3195.pth` | https://dl.fbaipublicfiles.com/segment_anything/sam_vit_l_0b3195.pth |
   | `vit_h` | `sam_vit_h_4b8939.pth` | https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth |
   | `sam3` | `sam3.pt` | https://huggingface.co/facebook/sam3 (gated; user must already have the file) |

3. Start and **leave the window open**:

```bat
uvicorn samapi.main:app --host 127.0.0.1 --port 8000 --workers 1
```

Smoke: `GET http://127.0.0.1:8000/sam/capabilities/` should include `"service": "sam-workbench-api"` and `"interactive_sessions": true`.

4. Then install the plugin JAR from  
   https://github.com/siyuvision/qupath-sam-workbench/releases/download/v0.1.6/qupath-sam-workbench-0.1.6-all.jar  
   into `%USERPROFILE%\QuPath\v0.7\extensions`. QuPath URL: `http://localhost:8000/sam/`

## Do not

- Do not call Hugging Face / `hf auth login` to fetch weights. Copy `sam3.pt` to `%USERPROFILE%\.samapi\sam3\sam3.pt` if the user already has it.
- Do not use `--workers` other than `1`.
- Do not commit `.venv`, `dist/`, `*.pt`, `*.pth`, or portable Runtime folders. Pack scripts in `scripts/` are for maintainers building netdisk zips only.
- Do not `git push` to `upstream` (`ksugar/samapi`). Push only `origin` (`siyuvision/sam-workbench-api`).
- If port 8000 is busy, stop the other SAM process; do not start a second API.
- Interactive Edit session active → Prompt/SAM3 return **409**. Accept or cancel first.

## Portable Runtime (maintainers only)

`scripts/pack-windows-cuda.ps1` / `scripts/pack-windows-cpu.ps1` write `dist\SAM-Workbench-Runtime-Windows-*` (gitignored). End users of a pack run `一键启动.bat`; agents installing from GitHub should use the venv path above, not the pack.

## Tests

```bat
python -m unittest discover -s tests -v
```

No weight downloads. Contact: WeChat `image_analysis_4u` / `service@siyuvisionbbs.com`. Donate: https://github.com/siyuvision/qupath-sam-workbench/blob/main/docs/donate.md

---
> Source: [siyuvision/sam-workbench-api](https://github.com/siyuvision/sam-workbench-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
