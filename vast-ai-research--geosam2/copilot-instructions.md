## geosam2

> Guidance for AI coding agents (Claude, Cursor, Codex, etc.) working in this

# AGENTS.md

Guidance for AI coding agents (Claude, Cursor, Codex, etc.) working in this
repository. Human contributors are welcome to read it too.

## Project at a glance

GeoSAM2 lifts SAM2 from images to 3D meshes. Given a multi-view rendering of a
mesh and an interactive prompt (a point or a 2D mask) on one view, it
propagates a consistent segmentation across views and back-projects the result
to per-face 3D part labels.

This repository ships **inference code only**. There is intentionally no
training, fine-tuning, dataset, or evaluation code. Do not add any.

- **Language**: Python 3.10+ (tested on 3.12), Linux.
- **Frameworks**: PyTorch ≥ 2.3, Hydra/OmegaConf, trimesh, OpenCV, Blender 4+
  (only for rendering new meshes).
- **Model backbone**: SAM2 (Apache 2.0, derived from `facebookresearch/sam2`).
- **License**: Apache 2.0. Keep it that way.

## Repository layout

```
GeoSAM2/
├── inference.py                       # Multi-view 3D segmentation entry point
├── single_view_point_prompt_infer.py  # 2D mask from interactive point prompts
├── geosam2_render.py                  # Headless Blender script (run under `blender -b -P`)
├── scripts/run_example.sh             # End-to-end demo on the bundled example
├── sam2/                              # SAM2 backbone + GeoSAM2 modifications
│   ├── configs/geosam2.yaml           # Hydra config used at inference time
│   ├── csrc/connected_components.cu   # Optional CUDA op (built via setup.py)
│   ├── modeling/                      # Model definition (use `*_geosam2` variants)
│   └── sam2_video_predictor_geosam2.py
├── utils/                             # Project-specific helpers (mode_ext C++ op, vis, lifting)
└── example/sample_00..03/             # Bundled multi-view demo assets
```

Treat `sam2/` as a **vendored, lightly-modified** copy of Meta's SAM2.
Preserve the upstream copyright headers; add a "Modifications copyright …"
line when you change a Meta-derived file. Avoid stylistic churn there.

## Setup

```bash
python -m pip install -r requirements.txt
python -m pip install -e .          # builds the optional CUDA extension
# Skip the CUDA build if no nvcc is available:
GEOSAM2_BUILD_CUDA=0 pip install -e .

# Pretrained weights:
mkdir -p ckpt
huggingface-cli download VAST-AI/GeoSAM2 geosam2.pt --local-dir ckpt
```

## Run / smoke-test

```bash
# End-to-end demo (downloads checkpoints separately):
bash scripts/run_example.sh

# Single-view 2D mask from point prompts:
python single_view_point_prompt_infer.py \
  --data-root example/sample_00 \
  --view-idx 0 \
  --point-prompt-file example/sample_00/point_prompts_scale1.json \
  --output-dir outputs/sample_00/2d_seg

# 3D propagation from an existing mask:
python inference.py \
  --data-root example/sample_00 \
  --mask-path outputs/sample_00/2d_seg/mask_view0000.npy \
  --mask-view 0 \
  --output-dir outputs/sample_00/3d_seg
```

There is no formal test suite. Before submitting any non-trivial change, at
minimum run `python -m py_compile` on the files you touched and execute the
demo script above on a CUDA box.

## Coding conventions

- **English only.** All identifiers, comments, docstrings, log strings,
  filenames, and example data names must be English. No CJK, no transliterated
  jargon, no internal codenames (`tripo_*`, `aigc`, `dkv2`, etc.).
- **No private absolute paths.** Never hard-code `/mnt/pfs/...`,
  `/mnt/afs/...`, `~user/...`, internal hostnames, S3 buckets, or teammate
  usernames. CLI arguments must default to relative project paths or be
  required.
- **No training/eval code.** PRs that add training loops, optimizers,
  dataloaders, dataset classes, or eval metrics will be rejected. The existing
  `self.training` checks inside vendored SAM2 modules are PyTorch-mode flags
  and must be left intact.
- **No prebuilt binaries in the tree.** `_C.so`, `*.pyc`, `__pycache__/`,
  checkpoints, renders, and other build artifacts belong in `.gitignore`.
- **Comments explain *why*, not *what*.** Don't add narration like
  `# import os` or `# loop over frames`. The diff already shows that.
- **Style.** Follow the surrounding code; default to PEP 8 for new files.
  4-space indentation, snake_case for functions/variables, CamelCase for
  classes. Type hints are encouraged on new public functions.
- **Logging.** Plain `print(...)` is acceptable for CLI scripts; reserve
  `logging` for library code that may be imported elsewhere.

## Patterns specific to this codebase

- The Hydra config `sam2/configs/geosam2.yaml` instantiates
  `sam2.modeling.sam2_base_geosam2.SAM2Base`, **not** the upstream
  `sam2_base.py` (which has been removed). When extending the model, modify
  the `*_geosam2` variants and update the config alongside.
- `SAM2VideoPredictor.propagate_in_video_v2` uses a rotated processing order
  so that the seed view is processed first; `frame_idx_real` is the original
  video index, while `frame_idx` is the rotated index used to fetch features.
- The connected-components op (`sam2/csrc/connected_components.cu`) is
  optional. If it is unavailable, mask post-processing is skipped with a
  warning — do not raise.
- `utils/mode_ext.py` JIT-compiles a small C++ extension via
  `torch.utils.cpp_extension.load`. It needs a working C++ compiler with
  OpenMP at runtime; do not replace it with a Python loop without preserving
  performance.

## Doing a release

The default branch is `main`. The `release` branch was used to stage the
initial open-source cleanup. For subsequent releases:

```bash
git checkout main && git pull
git tag -a vX.Y.Z -m "GeoSAM2 vX.Y.Z"
git push origin vX.Y.Z
```

Bump the `version` field in `pyproject.toml` in the same commit as the tag.

## When in doubt

- Read the `LICENSE` and `NOTICE` files before adding third-party code.
- Mirror the directory structure above when adding new modules; don't create
  parallel namespaces.
- If a change touches both vendored SAM2 code and project-specific code, split
  it into two commits so the diff against upstream stays auditable.

---
> Source: [VAST-AI-Research/GeoSAM2](https://github.com/VAST-AI-Research/GeoSAM2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
