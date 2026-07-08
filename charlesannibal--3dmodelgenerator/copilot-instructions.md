## 3dmodelgenerator

> CLI pipeline that takes creature concept art (reference images) and produces textured 3D `.glb` models using:

# ModelGeneratorCLI — Project Context

## What This Project Does

CLI pipeline that takes creature concept art (reference images) and produces textured 3D `.glb` models using:
1. **Hunyuan3D** for mesh generation from images
2. **Triplanar/cylindrical projection** for texture baking onto the mesh
3. Quality acceptance checks before the model is accepted as output

Entry points: `generator.bat` / `generator.ps1` / `generate.sh`

Key source files:
- `src/model_generator_cli/pipeline.py` — main pipeline (mesh gen → bake → export)
- `src/model_generator_cli/acceptance.py` — texture quality gate (T-01 through T-05)
- `retexture_*.py` — per-creature manual texture fix scripts (workarounds)

---

## Active Texturing Issues

### Issue A — Texture Bake Script Dependency (CRITICAL)
**Location:** `src/model_generator_cli/pipeline.py:233-235`

The primary texture bake depends on an external script:
`modelGenerator/scripts/texture_bake_python.py`

If this script is missing or fails, the whole bake returns `(None, None, "Texture bake failed")` and cascades into further failures (lines 279-289). No automatic recovery exists beyond the CPU fallback.

### Issue B — Fragile CPU Fallback
**Location:** `pipeline.py:612-628`

When the primary bake fails, a CPU fallback runs `model_generator.cpu_texture.project_texture()` using a single image from `views`. Problems:
- Fails entirely if `views` is empty ("no in-memory views for CPU fallback")
- Only uses one view — produces low-coverage textures
- No PBR quality control

### Issue C — Single-View Texture Coverage Gaps
**Location:** `pipeline.py:563-582`

When only one reference image is available, it is used for all projection angles. Backside and undersides get filled with a neutral tan fallback color (`pipeline.py:205-212`). These areas consistently fail acceptance checks.

### Issue D — Texture Quality Acceptance Failures
**Location:** `acceptance.py:276-349`

Five checks that reject poor textures:
| Check | Condition | Threshold |
|-------|-----------|-----------|
| T-01  | >85% black pixels | fail |
| T-02  | >80% neutral gray | fail |
| T-03  | Color variance (std dev) < 8 | fail |
| T-04  | Single channel dominates >60% | fail |
| T-05  | Alpha mean < 0.3 (too transparent) | fail |

Projections from single-view or fallback bakes frequently fail T-02, T-03, T-04.

### Issue E — Manual Per-Creature Retexture Workarounds
**Files:** `retexture_empalynx.py`, `retexture_mantorment.py`, `retexture_pupplynx.py`, `retexture_worcomb.py`

Hardcoded per-creature scripts that manually fix textures post-pipeline. These exist because the pipeline cannot reliably texture certain creature shapes. Not scalable.

---

## texture-hell — Active Texture Model

**Location:** `D:/Projects/Games/texture-hell/texture-hell_v1.0.safetensors`

SD2.1 768 fine-tune (Nazzaroth2/texture-hell) trained on **350 CC0 albedo textures from PolyHaven**. Broader material coverage than texture-diffusion (includes metals and fabric). Loaded via `from_single_file()`.

**Prompt format:** comma-separated tags, trigger token is `"texture"` (not `"pbr"`).
```python
from diffusers import StableDiffusionPipeline

pipe = StableDiffusionPipeline.from_single_file(
    "D:/Projects/Games/texture-hell/texture-hell_v1.0.safetensors",
    torch_dtype=torch.float16
).to("cuda")

image = pipe("texture, matte plastic, miniature figurine, seamless, flat lighting").images[0]
```

**Chosen because:** creature models have a miniature/plastic aesthetic — hard surface textures match better than organic ones.

**Override:** set env var `TEXTURE_DIFFUSION_MODEL=<path>` to swap models without code changes.

## texture-diffusion — Retired

**Location:** `D:/Projects/Games/texture-diffusion` (keep for reference, not used)
278 textures, SD2 base, `"pbr"` token — replaced by texture-hell.

---

## Planned Integration: texture-diffusion as Fallback

Replace the failing CPU fallback at `pipeline.py:612-628` with a three-tier chain:

1. **Primary:** Triplanar projection bake (existing)
2. **Fallback 1:** CPU single-view projection (existing)
3. **Fallback 2 (NEW):** Generate texture via `texture-diffusion` using creature name as prompt

The diffusion-generated textures would naturally pass all T-xx acceptance checks because the model was trained on real, varied PBR textures.

**Inpainting use case:** After a partial bake, identify neutral-tan fallback areas with a mask and use `StableDiffusionInpaintPipeline` to fill those regions before acceptance checks.

**Integration points:**
| File | Lines | Change |
|------|-------|--------|
| `pipeline.py` | 612-628 | Add diffusion fallback after CPU fallback |
| `pipeline.py` | 279-289 | Optionally redirect bake failure to diffusion |
| `acceptance.py` | 276-349 | No change needed — diffusion output passes all checks |
| `retexture_*.py` | all | Replace with parameterized diffusion call |

---

## Dependencies

- `diffusers>=0.21.0`
- `torch` with CUDA (already required for Hunyuan3D)
- Local model path: `D:/Projects/Games/texture-diffusion`

---
> Source: [CharlesAnnibal/3DmodelGenerator](https://github.com/CharlesAnnibal/3DmodelGenerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
