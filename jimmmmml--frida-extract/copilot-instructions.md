## frida-extract

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Standalone re-export of two pieces of the [FRIDA](https://github.com/cmubig/Frida) painting project (CMU, ICRA 2023/2024), decoupled from its robot/camera/ROS/audio/face/SD-loss stack:

1. **`complete_sketch`** — CoFRIDA Instruct-Pix2Pix: partial sketch + prompt → completed 512×512 PIL image.
2. **`sketch_to_vectors`** — FRIDA's differentiable stroke planner: sketch image → list of brush-stroke dicts in canvas-normalized `[0, 1]` coordinates. ML-based, ~35 min/run on RTX 6000 Ada.
3. **`sketch_to_vectors_fast`** — pure-CV skeleton tracer: same schema as (2), no ML, no GPU, ~0.2 s/run. Target for vehicle-based robot deployment.

The source project was extracted from `/data1/jl14087/Frida` so a downstream vehicle-based painting robot can consume stroke vectors without pulling in `rospy`, `xarm`, `gphoto2`, `pyrealsense2`, `librosa`, `whisper`, or SAM. Robot execution is **deliberately out of scope** — this repo returns vectors and stops.

Python 3.8 + CUDA (float16 inference for stage 1; stage 2 fast path is CPU). No test suite.

## Running

The smoke test (also the canonical example) is `examples/example_pipeline.py` — it chains `complete_sketch` → `sketch_to_vectors_fast` → `render_strokes_preview` and prints per-stage timing (end-to-end ≈ 15 s on a warm RTX 6000 Ada):

```bash
pip install -r requirements.txt
# If the default torch wheel doesn't match local CUDA, reinstall:
#   pip3 install torch torchvision --index-url https://download.pytorch.org/whl/cu118

HF_HUB_OFFLINE=1 TRANSFORMERS_OFFLINE=1 CUDA_VISIBLE_DEVICES=<free-gpu> \
  python examples/example_pipeline.py   # expects examples/partial_sketch_cat.jpg
```

A first CoFRIDA call downloads `skeeterman/CoFRIDA-Sharpie` from Hugging Face; the pipeline is then module-level-cached by `(model_id, device, dtype)`. The Sharpie repo ships **only** `pytorch_lora_weights.bin` (no safetensors, no `config.json`) — only the `load_attn_procs` branch in `_vendored/cofrida.py` succeeds; the `unet.from_pretrained` fallback's `OSError: ... no config.json` is expected noise.

**`HF_HUB_OFFLINE=1` matters.** Without it, diffusers does a synchronous HF-hub metadata probe that sometimes hangs on `sock.connect` even when files are cached. Set both flags once the model is on disk.

**Detached/long runs.** If you launch via `run_in_background` or a harness that manages subprocess trees, the python process can be SIGKILL'd mid-run. Use `setsid nohup bash -c '...' </dev/null >log 2>&1 & disown` so the process reparents to init (ppid=1) and survives. Especially relevant for `sketch_to_vectors` (ML path, ~35 min).

`examples/example_fast_vectorize.py` is a stage-2-only smoke test that reuses `completed.png` and sweeps three presets (`default` / `straight` / `aggressive`) so you can tune the CV vectorizer without re-running stage 1.

## Architecture

Four public entry points in `frida_extract/`, all re-exported from `__init__.py`:

- **`sketch_completion.py`** — thin wrapper over vendored `cofrida.get_instruct_pix2pix_model`. Resizes any input to 512×512 and calls the Instruct-Pix2Pix diffusion pipeline. Pipeline cached in a module-scope dict so repeat calls don't reload weights.
- **`sketch_to_vectors.py`** — ML path. Bypasses FRIDA's `Painter`/`Robot`/camera entirely. Builds a `types.SimpleNamespace` "opt" object populated from the cache's `stroke_settings_during_library.json`, calls `load_objectives_data` → `random_init_painting` → `optimize_painting`, then iterates `painting.brush_strokes` to emit dicts. The SimpleNamespace trick is what makes it possible to skip `src/options.py` and `materials.json` merging entirely — the planner only reads ~20 attributes and we provide all of them.
- **`sketch_to_vectors_fast.py`** — pure-CV path. Otsu-binarize → `skimage.skeletonize` → spur-prune → edge-based graph walk with tangent continuation at junctions → Gaussian smooth (sigma=1.5) → Douglas-Peucker simplify → split each polyline into arcs of length ≤ `MAX_STROKE_LENGTH` (from the cache's JSON) → fit `(x, y, rotation_rad, length_m, bend_m)` per arc. Emits the exact same dict schema as `sketch_to_vectors` so the downstream robot planner is unchanged. Runs in ~0.2 s on CPU. `thickness` is a single constant kwarg (default `0.5`) — no per-stroke width variation, because the vehicle robot uses one marker width.
- **`pipeline.py`** — `sketch_pipeline` chains `complete_sketch` + the ML `sketch_to_vectors`. The canonical `examples/example_pipeline.py` uses `sketch_to_vectors_fast` instead (bypasses `sketch_pipeline`).

### When to use which vectorizer

- **`sketch_to_vectors_fast`** (default for deployment). Vehicle-scale robot that can draw bent strokes; single marker thickness; wall-clock matters. End-to-end stage-1 + stage-2 ≈ 15 s.
- **`sketch_to_vectors`** (ML). Multi-brush, per-stroke thickness/color, differentiable-render-backed fidelity; no wall-clock budget. ~35 min on RTX 6000 Ada at default settings.

The CV path was added because stage-2 ML planning was ~200× slower than the <30 s target for the vehicle robot. If you change the fast path, keep the dict schema byte-identical to the ML path — downstream code branches on `cache_dir` contents, not on which function produced the dicts.

### Vendoring model (important)

`frida_extract/_vendored/` is a pruned copy of FRIDA's `src/`. FRIDA's modules use **flat, unqualified imports** (`from painting import Painting`, `from losses.clip_loss import ...`), so `_vendored/__init__.py` prepends its own directory to `sys.path` at import time. That's why every public module has `import frida_extract._vendored  # noqa: F401` at the top — side-effect import.

**Do not rewrite vendored imports to be package-qualified.** It would work but defeats the goal of keeping the vendored files byte-identical to upstream, which makes future re-syncs from FRIDA trivial.

### The one upstream patch

`_vendored/painting_optimization.py` has been patched to **lazy-import** the heavy loss backends (`audio_loss`, `emotion_loss`, `face`, `stable_diffusion`, `speech2emotion`). The top-level imports were moved into their respective `elif` branches inside `parse_objective` / `load_objectives_data`. This is what keeps `librosa`, `whisper`, SAM, etc. out of `requirements.txt`. If you re-sync from FRIDA, you must re-apply this patch.

`_vendored/paint_utils3.py` also has one unused import stripped (`from clip_attn.clip_attn import get_attention` — only `plan_hci.py` actually calls it, and we don't vendor that).

### Coordinate frame (deliberate deviation from FRIDA)

`sketch_to_vectors` returns `x, y ∈ [0, 1]` where `(0, 0)` is bottom-left. FRIDA's own `paint.py` applies `y = 1 - y` before sending to the robot; **we do not apply that flip**. Downstream consumers pick their own canvas orientation. This is called out in the README and the `sketch_to_vectors` docstring — don't "fix" it by re-adding the flip.

The remaining fields (`length_m`, `bend_m`, `thickness`, `alpha_rad`, `color_rgb`) are the raw values FRIDA's `BrushStroke` learns; they're meters/radians per the stroke library that trained `param2img.pt`.

### Cache dir contract

`sketch_to_vectors(cache_dir=...)` must point at a directory containing exactly three files:

- `param2img.pt` — the trained differentiable stroke renderer
- `param2stroke_settings.json` — renderer settings
- `stroke_settings_during_library.json` — physical envelope (`MAX_BEND`, `MIN_STROKE_Z`, `CANVAS_WIDTH_M`, etc.)

`caches/sharpie_short_strokes/` ships with the repo (single-color Sharpie ink). To swap brushes: retrain in the original FRIDA repo (`Frida/src/caches/`), drop the three files into a new subdir, point `cache_dir=` at it. For multi-color output, pass `ink=False` (strokes get optimizable RGB).

## Things that bite

- **Don't modify vendored files casually.** The only sanctioned patches are the lazy-import patch in `painting_optimization.py` and the stripped import in `paint_utils3.py`. Anything else breaks the "resync from upstream" workflow.
- `optimize_painting` writes TensorBoard events unconditionally. `sketch_to_vectors` creates a temp dir if `tensorboard_dir=None` — don't remove that fallback, the upstream code will crash without a writer.
- `load_objectives_data` only accepts filesystem paths, so PIL/ndarray sketches are materialized to a temp PNG inside `_normalize_sketch_to_path`.
- `Pillow==9.5.0`, `scipy==1.9.1`, `protobuf~=3.20`, `tensorboardX==2.1` are pinned — newer versions break FRIDA's vendored code. Leave the pins alone unless you're willing to test end-to-end.
- `diffusers==0.30.3` + `peft==0.13.2` are also pinned. **Don't bump.** Newer diffusers (0.36+) requires `peft>=0.17`, which in turn requires Python ≥ 3.9 — this repo is stuck on 3.8 via the FRIDA vendored code, so those are the ceilings. If pip-resolver complains about peft, the fix is *not* to upgrade.
- CUDA out of memory on the ML planner usually means `num_strokes × render_height` is too large. Lower `render_height` (default 256) before dropping strokes.
- `sketch_to_vectors_fast` splits each polyline at `MAX_STROKE_LENGTH` (from the cache JSON; 3 cm for Sharpie). Long curves (apple outline ≈ 25 cm) become 8-9 arcs — that's by design so the robot's trained-arc executor stays in its envelope. If the robot can draw longer strokes, raise that by overriding the cache JSON; don't edit the splitter.
- `render_strokes_preview` is an eyeball tool only (quadratic Bezier per stroke). It doesn't match FRIDA's trained renderer; don't use it for fidelity comparisons against `sketch_to_vectors`.

---
> Source: [JimmmmmL/frida-extract](https://github.com/JimmmmmL/frida-extract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
