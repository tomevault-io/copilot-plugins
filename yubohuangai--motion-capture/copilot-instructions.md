## motion-capture

> Development conventions for apps/reconstruction/ — multi-view object reconstruction scripts


# Reconstruction scripts (`apps/reconstruction/`)

## What this is

Incremental multi-view object reconstruction pipeline. 11 calibrated RGB cameras (EasyMocap format), foreground masks, COLMAP-based sparse/dense, then classical geometry → triangle mesh + texture. See the full step-by-step plan in **`/Users/yubo/github/ReconViaGen/.cursor/rules/incremental-recon-pipeline.mdc`**.

## Reference codebases — USE THESE when implementing

### 3D_Object_Reconstruction (`/Users/yubo/github/3D_Object_Reconstruction`)
Classical RGB-D multi-view reconstruction (Open3D). Borrow patterns for:
- **Point cloud cleanup**: `preprocess_pcd.py` — plane removal, statistical outlier removal, DBSCAN clustering
- **Surface reconstruction**: Open3D Poisson / ball-pivoting patterns in `registration.py`, `utils.py`
- **Pose graph optimization**: `pose_graph_ICP.py`, `pose_graph_Feature_based.py`
- **Caveat**: That repo assumes RGB-D + RealSense intrinsics. Adapt I/O — do not copy depth-sensor paths.

### ReconViaGen classical_mvs (`/Users/yubo/github/ReconViaGen/classical_mvs/`)
Custom plane-sweep MVS + TSDF fusion modules. Written for this user's camera setup but **untested / produced bad results as a monolith**. Individual modules may still be useful as reference:
- `camera_io.py` — EasyMocap camera loading, undistortion, mask loading
- `plane_sweep.py` — PyTorch GPU plane-sweep depth estimation (fallback if COLMAP dense MVS is inadequate)
- `tsdf_fusion.py` — Open3D ScalableTSDFVolume integration
- `texturing.py` — vertex color reprojection, UV texture baking
- `o3d_utils.py` — Open3D point cloud cleaning utilities

**Always inspect these files before writing similar functionality from scratch.**

## Camera / data conventions

- **Calibration**: `intri.yml` + `extri.yml` (EasyMocap format) at data root
- **Camera loading**: `easymocap.mytools.camera_utils.read_camera(intri_path, extri_path)` → dict with `K`, `R`, `T`, `dist` per camera name + `basenames` list
- **Images**: `<data_root>/images/<cam>/<frame:06d>.jpg`
- **Masks**: `<data_root>/masks/<cam>/<frame:06d>.png` (binary, 255 = foreground)
- **COLMAP workspace**: `<data_root>/colmap_ws/` (default from `export_colmap.py`) containing `sparse/0/`, `images/`, `masks/`, `database.db`

## CLI conventions (follow existing scripts)

- Positional arg or `--data` for data root / input path
- `--output` / `-o` with sensible default derived from input (e.g. `<data>/colmap_ws`)
- `argparse.BooleanOptionalAction` for on-by-default booleans (`--gpu` / `--no-gpu`)
- Print `[script_name]` prefixed status lines with counts, bounding boxes, timings
- Each script writes inspectable intermediate files (`.ply`, `.npy`, colormapped `.png`)

## Development rules

- **ONE script per session.** Implement, then STOP for user to test on server (Amax41, see ReconViaGen dev-server rule).
- **Refer to reference repos** (above) before writing Open3D / geometry code from scratch.
- **Fail loudly** — validate inputs exist and have sane values at the top of `main()`.
- **Do not update `RECONSTRUCTION_PIPELINE.md`** during implementation. Update only when the user asks, after a step is confirmed working.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yubohuangai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
