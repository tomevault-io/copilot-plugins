## tarotcardproject

> Generates 3D character models from each card for AR/VR use. Uses existing card images as input: background removal → 3D mesh generation → format conversion.

# Tarot Card Project

A tarot card layout, spread, and reader system. Differs from Rider-Waite by having more cards, compatible with both I-Ching balance model and Tarot action model.

## Tools & Stack

- **ComfyUI** — image and video generation workflows
- **n8n** — orchestration and automation
- **Perplexity** — research
- **TypeScript**

## Directory Structure

```
ComfyUI/           ComfyUI GUI workflow JSONs (Card_Dealer_*.json, Make_Images_For_Card.json, etc.)
n8n/               n8n orchestration workflow JSONs (Generate Dealer Animations.json, Generate Card Tarot Images.json, etc.)
data/              TarotSpreadsheet2.ods (card definitions)
Perplexity/        Research docs
```

## Data Paths

- Card images: `D:/data/cards/Standard/`
- Card back image: `D:/data/cards/Standard/IMAGE_Deck_Back.png`
- Dealer animations output: `D:/data/Dealer_Animations/`
- Tarot spreadsheet: `data/TarotSpreadsheet2.ods`

## Executables

| Tool | Path | Notes |
|------|------|-------|
| ffmpeg | `D:/ComfyUI_windows_portable/ffmpeg-8.0.1-full_build/bin/ffmpeg.exe` | Video concat, format conversion |
| n8n | `D:/n8n/` (run via `npm start`) | Orchestration; installed as npm dep in `D:/n8n/package.json` |
| Node.js | `C:/Program Files/nodejs/node.exe` | Runtime for n8n |
| Python | `C:/Users/justi/AppData/Local/Microsoft/WindowsApps/python.exe` | |
| ComfyUI | `D:/ComfyUI_windows_portable/ComfyUI/` | Image/video generation; API at `http://127.0.0.1:8188` |

## ComfyUI Installation

- **Path**: `D:/ComfyUI_windows_portable/ComfyUI/`
- **Hardware**: 6GB VRAM, 32GB system RAM
- **LoadImage input dir**: `D:/ComfyUI_windows_portable/ComfyUI/input/` (images must be copied here for LoadImage node)
- **API endpoint**: `http://127.0.0.1:8188/prompt` (POST), `http://127.0.0.1:8188/history/{prompt_id}` (GET)

### Custom Nodes Installed

| Node Pack | Status | Notes |
|-----------|--------|-------|
| ComfyUI-VideoHelperSuite | Working | VHS_VideoCombine for MP4 output |
| comfyui-rmbg | Working | BiRefNetRMBG for background removal (used by 3D pipeline) |
| comfyui-mixlab-nodes | Working | TripoSR nodes: `LoadTripoSRModel_`, `TripoSRSampler_`, `SaveTripoSRMesh` |
| FramePack-F1-T2V | Broken | No model loader produces FramePackMODEL type |
| FramePack-HY | No models | diffusers/ folder is empty, needs HunyuanVideo model download |

### Custom Nodes NOT Installed

- **AnimateDiff** (ComfyUI-AnimateDiff-Evolved) — do NOT generate workflows using AnimateDiff nodes

## Available Models (Wan 2.2 I2V)

All paths relative to `D:/ComfyUI_windows_portable/ComfyUI/`:

| Model | Path |
|-------|------|
| Wan 2.2 I2V high noise (14B fp8) | `models/diffusion_models/wan2.2_i2v_high_noise_14B_fp8_scaled.safetensors` |
| Wan 2.2 I2V low noise (14B fp8) | `models/diffusion_models/wan2.2_i2v_low_noise_14B_fp8_scaled.safetensors` |
| LightX2V 4-step LoRA (high noise) | `models/loras/wan2.2_i2v_lightx2v_4steps_lora_v1_high_noise.safetensors` |
| umt5_xxl text encoder (fp8) | `models/text_encoders/umt5_xxl_fp8_e4m3fn_scaled.safetensors` |
| CLIP-L | `models/text_encoders/clip_l.safetensors` |
| Wan 2.1 VAE | `models/vae/wan_2.1_vae.safetensors` |

## Critical Compatibility Notes

These were discovered through debugging — do not repeat these mistakes:

1. **AnimateDiff is NOT installed** — don't use AnimateDiff nodes in any workflow
2. **DualCLIPLoader does NOT support type "wan"** — valid types: sdxl, sd3, flux, hunyuan_video, hidream, hunyuan_image, hunyuan_video_15, kandinsky5, kandinsky5_image, ltxv, newbie
3. **CLIPLoader (single) DOES support type "wan"** — Wan only needs one text encoder (umt5_xxl), not two
4. **FramePack F1-T2V and FramePack HY are incompatible** — they use different types (FramePackMODEL vs FP_DIFFUSERS_PIPELINE), cannot interoperate
5. **Card back image filename**: `IMAGE_Deck_Back.png` (not `CardBack.png`)
6. **ComfyUI GUI JSON vs API JSON**: GUI format uses `nodes[]` and `links[]` arrays with numeric IDs; API format uses `prompt` object with string node IDs ("1", "2", ...) containing `class_type` and `inputs`
7. **`WanImageToVideo` does NOT accept `end_image`** — it only supports `start_image`. For start+end frame pinning (stop-motion interpolation), use **`WanFirstLastFrameToVideo`** instead. It accepts both `start_image` and `end_image` as optional inputs with the same interface, and works with the same Wan 2.2 I2V model.

## Dealer Animation Workflows (Wan 2.2 I2V)

Four animations: Cut, Fan, Merge, Rotate. All share the same node graph, differing only in prompts and output filenames.

### Node Chain

```
UNETLoader(1) → LoraLoaderModelOnly(8) → ModelSamplingSD3(9) → KSampler(10)
CLIPLoader(2) → CLIPTextEncode(4,5) → WanImageToVideo(7) → KSampler(10)
VAELoader(3) → WanImageToVideo(7) + VAEDecode(11)
LoadImage(6) → WanImageToVideo(7)
KSampler(10) → VAEDecode(11) → VHS_VideoCombine(12)
```

### Key Settings

- **KSampler**: steps=4, cfg=1.0, sampler=uni_pc_bh2, scheduler=sgm_uniform, denoise=1.0
- **WanImageToVideo**: width=480, height=832, length=81, batch_size=1
- **VHS_VideoCombine**: frame_rate=12, format=video/h264-mp4
- **ModelSamplingSD3**: shift=1.0 (flow matching)
- **LightX2V LoRA**: strength=1.0 (enables 4-step generation)

### n8n Orchestration

- Workflow: `n8n/Generate Dealer Animations.json`
- Pattern: Manual Trigger → Set Parameters → Build Animation List (Code) → HTTP POST to ComfyUI → Wait 300s → GET Status
- Each animation type gets its own prompt and output filename via template variables

---

## Workflow Design Philosophy

These principles apply to **all** n8n flows in this project. Follow them in every new flow and every refactor.

### Core Goal: Accessibility Across Skill Levels

This project is designed so that contributors need expertise in **only one area** to participate:

| Role | What they touch | What they don't need to know |
|------|----------------|------------------------------|
| **Artist / Prompt Engineer** | `Deck.json` — prompts, negative prompts, model names | n8n, ComfyUI API, code |
| **Technical Director** | `Deck.json` — geometry, paths, sampler settings | Art, prompt craft |
| **Workflow Developer** | n8n flow JSON | Art direction, specific model names |
| **ComfyUI Specialist** | ComfyUI workflow graph inside HTTP body | n8n, business logic |

### Design Principles

#### 1. Deck.json is the single source of truth
Every deck-specific value lives in `Deck.json`. No deck names, model names, paths, dimensions, prompts, or sampler settings are hardcoded in the n8n flow. This means the **same flow works for any deck** — just change `DeckJsonPath` in Set Global Parameters.

#### 2. Safe defaults — never hard-fail on optional sections
Optional sections (`Models`, `Samplers`) have built-in fallbacks so a deck author only needs to fill in the required sections (`Assets`, `Geometries`, `Prompts`). Example pattern:
```javascript
const models = deck.Models || {};
const checkpoint = models.KeyframeCheckpoint || 'dreamshaper_8.safetensors';
```

#### 3. Completeness — each flow is self-contained
Every flow must be importable and runnable with only `DeckJsonPath` changed. No hidden dependencies on data from other flows except through the Deck.json file.

#### 4. Sequential processing with self-contained Code nodes (not SplitInBatches loops)

**Do NOT use SplitInBatches (Loop Over Items) for ComfyUI submit→poll workflows.**
The "done" output of SplitInBatches sends the original input items — not the enriched loop-back items with fields added during the loop (e.g. `SavedFilename`, `SavedPath`). This causes the next stage to silently receive incomplete data.

**Use a single self-contained Code node** that loops over all jobs internally, submits each one, polls until done, and returns all results at once:

```javascript
// NOTE: require('axios') CRASHES the task runner — use require('http') instead
// See "HTTP request pattern" section in n8n Code Node Sandbox Restrictions below
const http = require('http');
const https = require('https');
// ... (add httpPost/httpGet helpers — see HTTP request pattern section)

const allJobs = $input.all().map(item => item.json);
const sleep = ms => new Promise(r => setTimeout(r, ms));
const results = [];

for (const job of allJobs) {
  // submit to ComfyUI
  const submitData = await httpPost(comfyUIAPI + '/prompt', { prompt: comfyPrompt });
  const promptId = submitData.prompt_id;

  // poll until done
  let done = false;
  for (let attempt = 0; attempt < 60 && !done; attempt++) {
    await sleep(3000);
    const historyData = await httpGet(comfyUIAPI + '/history/' + promptId);
    const entry = historyData[promptId];
    if (entry?.outputs?.['7']?.images?.length > 0) {
      results.push({ ...job, SavedFilename: entry.outputs['7'].images[0].filename });
      done = true;
    }
  }
  if (!done) throw new Error('Timeout: ' + job.OutputPrefix);
}
return results.map(r => ({ json: r }));
```

#### 5. Informative errors, not silent failures
Every Code node that can produce zero output must throw an Error with a message that tells the user exactly what to fix:
```javascript
if (jobs.length === 0) {
  throw new Error(
    '[Build Keyframe Job List] No jobs created. ' +
    'Check that AnimationTypes matches keys in Deck.json Prompts.'
  );
}
```
Warnings for skipped (but non-fatal) conditions use `console.warn()`.

#### 6. Insertable / overridable assets
Future flows should check for manually placed assets before generating. The convention:
- Named `KF_<AnimType>_<Index>.png` for keyframes
- Named `<AnimType>_back.png` for card backs
- Placed in the deck's `Assets.Directory` or `Assets.AnimationKeyframes` folder
- If found, skip generation and use the existing file

This lets an artist drop a hand-crafted image in place of a generated one without touching any code or flow.

### Deck.json Schema

#### Required
```json
{
  "Assets": {
    "Directory": "D:/data/cards/MyDeck",
    "AnimationKeyframes": "D:/data/cards/MyDeck/keyframes",
    "Cut":    "D:/data/cards/MyDeck/animations/cut.mp4",
    "Fan":    "D:/data/cards/MyDeck/animations/fan.mp4",
    "Merge":  "D:/data/cards/MyDeck/animations/merge.mp4",
    "Rotate": "D:/data/cards/MyDeck/animations/rotate.mp4"
  },
  "Geometries": {
    "Animation": { "Width": 512, "Height": 896, "FrameRate": 24, "InterpolationFrames": 25 }
  },
  "Prompts": {
    "Cut": {
      "Negative": "bad quality, blurry, distorted",
      "Keyframes": ["frame 1 positive prompt", "frame 2 positive prompt"]
    }
  }
}
```

#### Optional (safe defaults built-in)
```json
{
  "Models": {
    "KeyframeCheckpoint": "dreamshaper_8.safetensors",
    "InterpolationUNet":  "wan2.2_i2v_high_noise_14B_fp8_scaled.safetensors",
    "InterpolationCLIP":  "umt5_xxl_fp8_e4m3fn_scaled.safetensors",
    "InterpolationVAE":   "wan_2.1_vae.safetensors"
  },
  "Samplers": {
    "Keyframe":      { "Steps": 20, "CFG": 7.0, "Denoise": 1.0, "Sampler": "dpmpp_2m",    "Scheduler": "karras"      },
    "Interpolation": { "Steps": 20, "CFG": 7.0, "Denoise": 1.0, "Shift":  3.0, "Sampler": "uni_pc_bh2", "Scheduler": "sgm_uniform" }
  }
}
```

### n8n Data Flow Conventions

- After `Extract from File` (fromJson), parsed content is under `.data`: `$input.first().json.data`
- Cross-node references use `.first()` for single items: `$('NodeName').first().json`
- Accumulate all items with `$input.all()` at aggregation points (Build Interpolation Job List, Build FFmpeg Concat Commands)
- Job items carry **all** fields needed downstream — avoid repeated cross-node references inside loops
- ComfyUI Submit nodes output only `{prompt_id, number, node_errors}` — merge with job data in the Capture node

---

## n8n Code Node Sandbox Restrictions (v2.7.5)

These apply to **all Code nodes** in n8n. Test environment: n8n 2.7.5, self-hosted on Windows.

### What is blocked by default

| API | Error | Status |
|-----|-------|--------|
| `fetch()` | `fetch is not defined` | Blocked — global not available |
| `require('http')` | `Module 'http' is disallowed` | Blocked — **unless** `NODE_FUNCTION_ALLOW_BUILTIN=*` is set |
| `require('https')` | `Module 'https' is disallowed` | Blocked — **unless** `NODE_FUNCTION_ALLOW_BUILTIN=*` is set |
| `require('axios')` | Task runner crash / `TypeError: Cannot assign to read only property 'toString'` | **BROKEN** — `preventPrototypePollution()` freezes `FormData.prototype`; axios's `form-data` dep crashes on load |
| `new URL(str)` | `URL is not defined` | Blocked — not in vm context's `getNativeVariables()` |
| `$helpers.httpRequest()` | `$helpers is not defined` | Not available in Code nodes (only in custom node SDK) |

### What works

| API | How to enable | Notes |
|-----|---------------|-------|
| `require('http')` | Set `NODE_FUNCTION_ALLOW_BUILTIN=*` | **Use this instead of axios** for HTTP calls; unaffected by frozen prototypes |
| `require('https')` | Set `NODE_FUNCTION_ALLOW_BUILTIN=*` | Same as http for HTTPS |
| `require('url')` | Set `NODE_FUNCTION_ALLOW_BUILTIN=*` | Provides `URL` class if needed (or use regex parsing — see below) |
| `require('fs')` | Set `NODE_FUNCTION_ALLOW_BUILTIN=*` | File read/write/copy operations |
| `require('path')` | Set `NODE_FUNCTION_ALLOW_BUILTIN=*` | Path joining |
| `require('child_process')` | Set `NODE_FUNCTION_ALLOW_BUILTIN=*` | Run shell commands (replaces Execute Command node) |

### HTTP request pattern (use instead of axios)

```javascript
const http = require('http');
const https = require('https');

const httpRequest = (method, urlStr, body) => new Promise((resolve, reject) => {
  // Parse URL manually — new URL() is NOT available in vm context
  const m = urlStr.match(/^(https?):\/\/([^:/]+)(?::(\d+))?(\/?[^?]*)(\?.*)?$/);
  const protocol = m[1], hostname = m[2];
  const port = m[3] ? parseInt(m[3]) : (protocol === 'https' ? 443 : 80);
  const path = (m[4] || '/') + (m[5] || '');
  const lib = protocol === 'https' ? https : http;
  const options = { hostname, port, path, method, headers: {} };
  if (body) {
    options.headers['Content-Type'] = 'application/json';
    options.headers['Content-Length'] = Buffer.byteLength(body);
  }
  const req = lib.request(options, (res) => {
    let data = '';
    res.on('data', (chunk) => { data += chunk; });
    res.on('end', () => { try { resolve(JSON.parse(data)); } catch(e) { resolve(data); } });
  });
  req.on('error', reject);
  if (body) req.write(body);
  req.end();
});
const httpPost = (url, bodyObj) => httpRequest('POST', url, JSON.stringify(bodyObj));
const httpGet  = (url)          => httpRequest('GET',  url, null);
```

**Why not axios?** n8n's task runner runs `preventPrototypePollution()` at startup, which calls `Object.freeze(fn.prototype)` on every global function including `FormData`. When `require('axios')` is called, its bundled `form-data` dependency immediately tries to set `FormData.prototype.toString`, which throws `TypeError: Cannot assign to read only property 'toString'`. This crashes the task runner process, producing the generic "Node execution failed" error with ~150ms timing. The Node.js built-in `http`/`https` modules are unaffected.

### Required n8n startup flags

```cmd
set NODE_FUNCTION_ALLOW_BUILTIN=* && set NODE_FUNCTION_ALLOW_EXTERNAL=* && set N8N_RUNNERS_TASK_TIMEOUT=28800 && n8n start
```

| Flag | Purpose | Default | Required value |
|------|---------|---------|---------------|
| `NODE_FUNCTION_ALLOW_BUILTIN` | Allows `fs`, `path`, `child_process` etc. in Code nodes | restricted | `*` |
| `NODE_FUNCTION_ALLOW_EXTERNAL` | Allows npm packages like `axios` in Code nodes | restricted | `*` |
| `N8N_RUNNERS_TASK_TIMEOUT` | Max seconds a single Code node task may run | **300** | `28800` (8 hrs) |

**Why 28800?** Wan 2.2 I2V (14B fp8) on 6GB VRAM takes 10–30 min per clip. 8 interpolation pairs × 30 min = 240 min worst case. The Code node runs ALL clips in one task, so the task timeout must cover the full batch (28800s = 8 hrs).

**Common mistakes:**
- `ODE_FUNCTION_ALLOW_EXTERNAL` (missing leading `N`) — silently ignored
- Single quotes in Windows `set`: `set FOO='bar'` sets value to `'bar'` including the quotes

### Execute Command node is unavailable

`n8n-nodes-base.executeCommand` is unrecognized in this n8n 2.7.5 setup. **Replace with a Code node using `child_process`:**

```javascript
const { execSync }  = require('child_process');
const fs            = require('fs');
const path          = require('path');

// Write any input files needed
fs.mkdirSync(path.dirname(outputPath), { recursive: true });
fs.writeFileSync(inputFilePath, content, 'utf8');

// Run the command
try {
  execSync(`ffmpeg -y -f concat -safe 0 -i "${listPath}" -c copy "${outputPath}"`,
           { encoding: 'utf8' });
} catch (e) {
  throw new Error('Command failed: ' + (e.stderr || e.message));
}
```

---

## Generate Dealer Stop Motion — Current Architecture

```
Set Global Parameters
  └─ Read Deck.json → Parse Deck.json → Build Keyframe Job List (N items)
       └─ Generate All Keyframes (Code: axios loop, submit+poll each job)
            └─ Build Interpolation Job List
                 └─ Generate All Interpolations (Code: axios+fs loop, copy+submit+poll each pair)
                      └─ Build FFmpeg Concat Commands
                           └─ FFmpeg Concat Clips (Code: fs+child_process, write list+run ffmpeg)
                                └─ Report Results
```

---

## Generate Card 3D Objects — Architecture

Generates 3D character models from each card for AR/VR use. Uses existing card images as input: background removal → 3D mesh generation → format conversion.

### Pipeline Stages

1. **Load existing card image** — `CARDIMG_*_upright.png` for upright, `CARDIMGUPRIGHT_*_<orientation>.png` for reversed/between
2. **Generate back-view image** — DreamShaper 8 txt2img, composing prompts from CharacterImage, Keywords, Music.Tags, and Narration.Tone. Supports insertable assets (skip if `BACKVIEW_*` already exists).
3. **Remove background** — BiRefNetRMBG (RMBG-2.0), produces clean RGBA
4. **Generate 3D mesh** — TripoSR (image-to-3D), outputs GLB with albedo textures
5. **Refine 3D mesh** — trimesh post-processing: smooth normals, decimate, clean degenerate faces
6. **Upscale textures** — Pillow lanczos 2x on GLB albedo textures (currently a graceful no-op: TripoSR outputs vertex colors, not UV-mapped textures; will activate after SPAR3D upgrade)
7. **Convert GLB to OBJ** — Python trimesh via `execFileSync`, produces OBJ+MTL

### n8n Workflow

```
Set Global Parameters (DeckJsonPath, TestCardFilter, Orientations, TripoSRResolution, SkipExisting)
  └─ Read Deck.json → Extract from File
       └─ Build 3D Job List (Code: reads card JSONs, checks for existing GLB, builds job array)
            └─ Generate Back-View Images (Code: DreamShaper 8 txt2img, skip if exists)
                 └─ Generate All 3D Models (Code: http loop, submits chained ComfyUI workflow, polls)
                      └─ Refine 3D Meshes (Code: trimesh smooth/decimate/clean)
                           └─ Upscale Textures (Code: Pillow lanczos 2x on GLB albedo textures)
                                └─ Convert GLB to OBJ (Code: trimesh GLB→OBJ conversion)
                                     └─ Report Results
```

### ComfyUI Node Chain — 3D Generation (single prompt per card)

```
LoadImage(1) → BiRefNetRMBG(2) → SaveImage(3) [bg-removed ref]
BiRefNetRMBG(2) → TripoSRSampler_(5) [image only, no mask]
LoadTripoSRModel_(4) → TripoSRSampler_(5) → SaveTripoSRMesh(6)
```

### ComfyUI Node Chain — Back-View Generation (DreamShaper 8 txt2img)

```
CheckpointLoaderSimple(1) → CLIPTextEncode(2,3) → KSampler(5) → VAEDecode(6) → SaveImage(7)
EmptyLatentImage(4) → KSampler(5)
```

### File Naming

| File | Purpose |
|------|---------|
| `BACKVIEW_{CardSlug}_{orientation}.png` | Back-view reference image |
| `3DREF_{CardSlug}_{orientation}.png` | Background-removed source image |
| `MODEL_{CardSlug}_{orientation}.glb` | 3D mesh (GLB, primary) |
| `MODEL_{CardSlug}_{orientation}.obj` | 3D mesh (OBJ, secondary) |

### Key Parameters (from Deck.json `Samplers.3D`)

- **Resolution**: 768 (TripoSR mesh resolution). Range: 128–12288.
- **Threshold**: 15.0 (mesh density / marching cubes threshold)
- **ChunkSize**: 16384 (TripoSR model chunk size)
- **BackViewSteps**: 25, **BackViewCFG**: 7.0 (DreamShaper 8 sampler settings)
- **BiRefNetRMBG model**: BiRefNet-general, `refine_foreground: true`

### Prerequisites

- TripoSR model: `models/triposr/model.ckpt` (~1.6GB, download from `stabilityai/TripoSR` on HuggingFace)
- Python `trimesh` package: `pip install trimesh` for GLB→OBJ conversion
- ComfyUI custom nodes already installed: `comfyui-rmbg`, `comfyui-mixlab-nodes` (TripoSR)

### Critical Compatibility Notes (3D)

8. **TripoSR node names have trailing underscores**: `LoadTripoSRModel_` and `TripoSRSampler_` (not `LoadTripoSRModel` / `TripoSRSampler`)
9. **TripoSRSampler_ mask input causes `Cannot handle this data type` error** — pass only the `image` output from BiRefNetRMBG (output index 0), do NOT pass the `mask` (output index 1)
10. **BiRefNetRMBG requires explicit optional params** — set `mask_blur: 0`, `mask_offset: 0`, etc. to avoid `'mask_blur'` KeyError

### 3D Refinement Pipeline

Post-generation mesh cleanup applied in the "Refine 3D Meshes" Code node:

1. **Smooth normals** — Laplacian smoothing to reduce TripoSR staircase artifacts
2. **Decimate** — Reduce polygon count while preserving shape (target: ~50k faces)
3. **Clean degenerate faces** — Remove zero-area triangles, non-manifold edges, duplicate vertices
4. **Texture upscaling** — Pillow lanczos 2x on extracted albedo textures (graceful no-op with TripoSR vertex colors; future: ComfyUI RealESRGAN after SPAR3D upgrade)

Artist override: drop a hand-refined `MODEL_*_*.glb` into the card directory and set `SkipExisting=true` to bypass regeneration.

### Future: SPAR3D Upgrade

**Evaluation result: Not viable for current 6GB VRAM hardware.**

SPAR3D (Stable Point-Aware Reconstruction of 3D) is the successor to TripoSR in the lineage: TripoSR → SF3D (Stable Fast 3D) → SPAR3D. It improves on TripoSR in several ways:

| Feature | TripoSR (current) | SPAR3D |
|---------|-------------------|--------|
| Backside quality | Poor (hallucinated) | Good (point cloud conditioned) |
| Surface smoothness | Staircase artifacts (Marching Cubes) | Smoother (Marching Tetrahedron) |
| Texturing | Vertex colors only | UV-mapped with proper materials |
| Point cloud editing | No | Yes (Gradio demo or external tools) |
| Remeshing | None built-in | Triangle or Quad options with target face count |
| VRAM (default) | ~6 GB | ~10.5 GB |
| VRAM (low mode) | Adjustable via `chunk_size` | ~7 GB (`SPAR3D_LOW_VRAM=1`) |

**Why not now:** Default 10.5GB VRAM far exceeds our 6GB budget. Even low-VRAM mode (~7GB) won't fit. TripoSR + trimesh refinement is the best option for current hardware.

**Upgrade path:** If GPU is upgraded to 8GB+, install SPAR3D as a ComfyUI custom node:
- Clone `Stability-AI/stable-point-aware-3d` into `ComfyUI/custom_nodes/`
- Provides nodes: `SPAR3DLoader`, `SPAR3DSampler`, `SPAR3DPreview`, `SPAR3DSave`, `SPAR3DPointCloudLoader`, `SPAR3DPointCloudSaver`
- Update `Deck.json` `Models.3D` to reference SPAR3D checkpoint
- Update `Generate Card 3D Objects.json` ComfyUI prompt template to use SPAR3D node chain instead of TripoSR

---
> Source: [justice8096/TarotCardProject](https://github.com/justice8096/TarotCardProject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
