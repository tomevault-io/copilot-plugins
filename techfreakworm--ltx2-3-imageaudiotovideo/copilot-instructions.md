## ltx2-3-imageaudiotovideo

> Generates lip-synced talking-head videos from a still portrait image and an audio file. Built on Lightricks' LTX-2 v2.3 — a 22B-parameter DiT-based audio-video foundation model. Takes an image, audio, and text prompt as input; outputs an MP4 where the person appears to speak the audio with synchronized lip movements.

# LTX-2.3 Image + Audio to Video (Lip Sync)

## Project Overview

Generates lip-synced talking-head videos from a still portrait image and an audio file. Built on Lightricks' LTX-2 v2.3 — a 22B-parameter DiT-based audio-video foundation model. Takes an image, audio, and text prompt as input; outputs an MP4 where the person appears to speak the audio with synchronized lip movements.

## Architecture

Two pipeline implementations wrapping the upstream `ltx-core` and `ltx-pipelines` packages:

### Two-Stage Pipeline (`a2vid_preloaded.py`) — Primary

`A2VidPreloaded` uses upstream `DiffusionStage` block classes (same as `a2vid_two_stage.py` in ltx-pipelines). All models lazy-load in `__call__` and free after use — managed by the block classes internally.

**Stage 1:** Denoise at half resolution (W/2 x H/2) for N steps. Audio frozen as conditioning.
**Spatial Upscale:** 2x latent upscaling via `VideoUpsampler` block (with MPS CPU fallback).
**Stage 2:** Refine at full resolution for 4 distilled steps with distilled LoRA. Audio frozen.

Audio is **frozen** (`frozen=True, noise_scale=0.0`) in both stages — same `encoded_audio_latent` tensor passed to both. Audio conditions the video but is never modified. Original waveform is muxed into the output MP4.

### Single-Stage Pipeline (`a2vid_one_stage.py`) — Fallback via `--pipeline single-stage`

`A2VidOneStage` uses block-based loading from `ltx_pipelines.utils.blocks`. Denoises directly at target resolution in one pass — no upsampling. Use as fallback if two-stage fails despite CPU upsampler fallback.

Models are loaded sequentially and freed after use (same pattern as two-stage):
1. Load PromptEncoder → encode prompts → `del` + flush (~24GB reclaimed)
2. Load AudioConditioner → encode audio → `del` + flush (~4GB reclaimed)
3. Load ImageConditioner → encode image → `del` + flush (~8GB reclaimed)
4. Load DiffusionStage (22B transformer) → denoise → `del` + flush (~44GB reclaimed)
5. Load VideoDecoder → decode → `del` + flush

### Audio Handling

Audio is NOT generated — it conditions the video. The audio latent is encoded once via the audio VAE, then "frozen" during diffusion (denoise_mask set to zeros). The original audio waveform is muxed back into the output MP4.

The audio VAE requires **stereo input**. Mono files are auto-converted by `ensure_stereo()` in `generate.py`.

## File Structure

```
generate.py            # CLI entry point — parses args, loads config, runs pipeline
a2vid_preloaded.py     # Two-stage pipeline (staged transformer loading, memory-efficient)
a2vid_one_stage.py     # Single-stage pipeline (MPS-compatible fallback)
config.yaml            # Quality config (10 steps, STG off, ~5 min)
config.fast.yaml       # Fast config (same values, for backward compat)
setup.sh               # Automated setup: init submodule, create venv, install deps
.gitignore             # Ignores .venv/, models/, outputs/
inputs/                # Input files (user-provided images and audio)
outputs/               # Generated MP4 videos (timestamped filenames)
demo/                  # Demo assets (pirate scene input/output for README)
models/                # Downloaded model files (not committed)
  gemma_root/          # Gemma text encoder + tokenizer files
LTX-2/                 # Git submodule: techfreakworm/LTX-2 (fork with MPS fixes)
  packages/ltx-core/   # Core model components (installed as editable package)
  packages/ltx-pipelines/ # Pipeline utilities (installed as editable package)
.venv/                 # Python 3.11 virtual environment (not committed)
FUTURE_IMPROVEMENTS.md # Known MPS limitations and plans to fix
```

## Setup

```bash
bash setup.sh
# Downloads/places models in models/ directory
# See README.md for model download links
```

## CLI Usage

```bash
# Two-stage (default) — ~25 min, best quality
python generate.py --image inputs/portrait.png --audio inputs/speech.wav --prompt "..."

# Single-stage fallback — ~66 min at 20 steps
python generate.py --pipeline single-stage --image inputs/portrait.png --audio inputs/speech.wav --prompt "..."

# Fast mode — ~10 min
python generate.py --config config.fast.yaml --image inputs/portrait.png --audio inputs/speech.wav --prompt "..."
```

`--pipeline` flag: `auto` (default, uses two-stage), `two-stage`, `single-stage`.

## Key Constraints

### Memory (128GB Apple Silicon)
- Both pipelines load/free models sequentially — only one large model in memory at a time
- Two-stage: models freed via `model.to("meta")` + `gc.collect()` + `torch.mps.synchronize()` + `torch.mps.empty_cache()`
- Single-stage: models freed via `del` + same flush pattern
- Resolution capped via `max_pixels` config (default 614,400 = ~640x960) to prevent OOM
- **NEVER set `max_pixels: 0`** — removes resolution cap and WILL OOM on 128GB with large images
- Peak memory: ~70GB (two-stage, stage 1 with shared models) / ~50GB (single-stage peak)
- Memory logging: both pipelines log RSS + device-allocated memory at each load/free checkpoint
- `generate.py` logs system memory at startup and warns if available < 70GB

### MPS Compatibility
- The spatial upsampler (einops rearrange + depthwise conv on 5D tensors) can crash on MPS
- `a2vid_preloaded.py` has automatic CPU fallback: if upsampler fails on MPS, it runs on CPU (<60s)
- If two-stage still fails, use `--pipeline single-stage` as fallback

### Frame Count (8k+1 Format)
The transformer's temporal architecture requires `num_frames = 8k + 1` (e.g., 9, 17, 25, ..., 121, ...).

### Resolution
All dimensions are snapped to multiples of 32. Default behavior matches input image resolution (capped by `max_pixels`).

### Audio
- Audio VAE requires **stereo** input. Mono auto-converted.
- Audio latent is frozen during diffusion — conditions video but is not modified.
- Audio is internally resampled to 16kHz by the AudioProcessor.
- Keep audio under 5 seconds for best lip sync results.

### Lip Sync (image_conditioning_strength)
- `image_conditioning_strength` controls how much the first frame locks to the input image.
- `1.0` = frame 0 completely frozen (denoise_mask=0.0) — **no lip sync possible**.
- `0.7` (default) = preserves identity while allowing lip sync (per ComfyUI workflows).
- `0.0` = no image conditioning — model generates freely (identity may drift).
- Tune via config or `--image-strength` CLI flag.
- Known LTX-2.3 limitation: A2V is better at energy/pacing than precise lip sync (community-wide).
- Best results: front-facing portrait, visible mouth, prompt with "speaking" / "lip sync" keywords.

## Config

Two configs shipped:
- `config.yaml` — Quality mode: 30 steps, upstream LTX-2.3 defaults, ~35 min (two-stage)
- `config.fast.yaml` — Fast mode: 10 steps, STG off (`stg_scale=0.0`), ~10 min (two-stage)

### Guidance Parameters (aligned with upstream `LTX_2_3_PARAMS`)

Both pipelines use `GuidedDenoiser` with constant guidance params throughout all denoising steps. Values are aligned with the official upstream defaults in `ltx-pipelines/utils/constants.py`:

- Video: cfg=3.0, stg=1.0, rescale=0.7, modality=3.0, stg_blocks=[28]
- Audio: cfg=7.0, modality=3.0

**Do NOT reduce audio_cfg below 7.0** — this is the primary lip sync driver. **stg_blocks must be [28] for LTX-2.3** (block 29 is the LTX-2.0 default).

### Speed Notes
- STG ON = 3-4 forward passes/step (cond + uncond + perturbed + modality). STG OFF = 2-3 passes.
- Two-stage is ~2.6x faster than single-stage: 30 steps at half-res + 4 steps at full-res vs 30 steps at full-res.
- Stage 2 always uses SimpleDenoiser (1 pass/step, no STG/CFG) regardless of config.

## Development Notes

- `generate.py` is the only CLI entry point. Both pipeline classes are importable modules.
- `LTX-2/` is a **git submodule** pointing to `techfreakworm/LTX-2` (fork with MPS fixes, branch `fix/mps-apple-silicon-compatibility`). Upstream PR: Lightricks/LTX-2#184.
- `ltx-core` and `ltx-pipelines` are editable installs from `LTX-2/packages/` — changes there take effect immediately.
- The audio guider uses `MultiModalGuiderParams()` (all defaults) — audio is frozen during diffusion and must NOT be guided. This matches the upstream `a2vid_two_stage.py` exactly.
- No tests or CI exist. Validate changes by running generation end-to-end.
- Python 3.11 is used in the venv.

---
> Source: [techfreakworm/LTX2.3-ImageAudioToVideo](https://github.com/techfreakworm/LTX2.3-ImageAudioToVideo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
