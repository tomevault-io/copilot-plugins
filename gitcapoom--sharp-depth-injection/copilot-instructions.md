## sharp-depth-injection

> Runtime extension to Apple's **ml-sharp** Gaussian-splat predictor that feeds a metric depth

# CLAUDE.md — Sharp_Depth_Injection

Runtime extension to Apple's **ml-sharp** Gaussian-splat predictor that feeds a metric depth
prior into SHARP at inference, plus a FastAPI **service** that turns `image (+ depth) → .ply`.
Built for a film / previz pipeline: 3D rough (e.g. Houdini) → image model (img2img) → SHARP splat.
**Consumers:** downstream apps call the HTTP service (e.g. a direct client, and a previz EXR/splat delivery tool).

## Layout
- `sharp_ext/` — the extension. **Hot-swapped into a built predictor at runtime; NEVER edit `../ml-sharp/`.**
  - `external_depth.py` — `ExternalDepthGaussianComposer` (composer subclass). Depth→grid resize is
    **max-pool of inverse depth** (= min-pool of depth = closest surface); `propagate_to_other_layers=True`;
    capture/lock hooks for 4DGS.
  - `swap_composer.py` — `install_external_composer` (the runtime hot-swap).
  - `predict_with_depth.py` — `predict_image_with_depth(...)`, the entry point; dispatches the 3 depth methods.
  - `depth_grade.py` — value-space grade curves (source `percentile`|`region`, curve `affine`|`polynomial`|`histogram`, `grade_min_slope` floor to stop 3DGS popping). Pure numpy/scipy.
  - `_predict_at_res.py`, `depth_io.py`, `frame_source.py`, `batch_4dgs.py`.
- `service/` — `serve.py` (FastAPI: `/generate`, `/inspect`, `/health`), `API.md`, `README.md`, `install.ps1`, `run.ps1`.
- `tests/` — self-consistency (Test 1), decoder stride (Test 2), depth methods/grade.
- sibling `../ml-sharp/` @ commit **`cdb4ddc6`** (`pip install -e`), untouched.
- `SINGLE_FRAME_INTEGRATION.md` — self-contained brief to embed the single-frame path in another app.

## Depth methods (`depth_method`)
- `sharp` — plain SHARP, no depth.
- `exr_pixel` — per-pixel inverse-depth blend of the EXR (`blend_alpha`). Anchors scale per pixel but
  **FIGHTS SHARP's geometry → flying / smeared gaussians at silhouettes. Do not use for splats.**
- `exr_grade` — remap SHARP's **own** predicted depth to match the EXR's distribution (monotone value
  curve). Preserves SHARP's coherent geometry → clean. **The right method for splats.**

**Working splat recipe** (reproduces known-good service output, per-cell exact): `exr_grade`,
`grade_source=region` (albedo-segmented, drops sky), `grade_curve=polynomial`, `grade_min_slope=1.0`,
`focal_mm`=⅔×plate focal, `aperture_mm`=sensor. Needs `depth` (metric CGdepth EXR) + `albedo`.

## The ~3.4× NDC scale offset (important)
The released checkpoint applies a fixed ~2.25× NDC scaling, so an injected metric depth `D` lands at
world-Z ≈ 3.4·D (both injection methods, uniform). A `metric_rescale` + `far_cap` fix was added and
then **REVERTED** (commit `fdd47f9` → `4685a07`): `metric_rescale` is a harmless *uniform* scale, but
`far_cap` moved far geometry onto a wall (flattening + wrong-distance blobs), and neither belongs in the
shared service (downstream consumers depend on it). **Do NOT re-add them.** Correct the scale *downstream*
in the consumer with a single uniform factor (divide positions + gaussian sizes by the measured `k`).

## Service ops
- Scheduled task **"SHARP Image2Splat"** runs `python service/serve.py` on `:8765` (CUDA, one model ≈ 6 GB VRAM held resident).
- Redeploy after a code change (non-interactive): `Stop-ScheduledTask -TaskName "SHARP Image2Splat"` then
  `Start-ScheduledTask -TaskName "SHARP Image2Splat"`, then poll `GET /health` until `model_loaded:true`.
- `/generate` returns raw PLY bytes, or JSON if `output_path` is set (server-side write; shared FS only).

## Gotchas (hard-won — don't relitigate)
- Decoder stride = **2** → gaussian grid **768²** (=1536/2); 2 layers → **1,179,648** gaussians.
- `internal_res` locked to **1536** (the SPN encoder only tiles cleanly there; 3072 fails at split).
- Install the **CUDA** torch build explicitly (`--index-url .../cu128`) or it silently runs on CPU.
- Depth must be camera-space **Z, metres, +forward** (not ray distance, not world-Z). Sky/no-hit = 0 → `load_depth_exr` remaps to a far value.
- Env: Python **3.13** venv at `.venv/`. Windows 11, PowerShell primary, RTX 4090.

## Repo
Public GitHub `gitcapoom/Sharp_Depth_Injection`, default branch `main`.

---
> Source: [gitcapoom/Sharp_Depth_Injection](https://github.com/gitcapoom/Sharp_Depth_Injection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
