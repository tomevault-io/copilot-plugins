## arbiter

> Arbiter is a unified GPU model server. It manages 11+ ML models on a single NVIDIA Grace Blackwell (128GB VRAM, 100GB budget). Key components:

# Arbiter — Development Guide

## Architecture

Arbiter is a unified GPU model server. It manages 11+ ML models on a single NVIDIA Grace Blackwell (128GB VRAM, 100GB budget). Key components:

- **`src/arbiter/server.py`** — FastAPI server (port 8400). Entry point.
- **`src/arbiter/scheduler.py`** — SJF scheduler. Picks jobs, loads models, dispatches to workers.
- **`src/arbiter/memory.py`** — GPU memory manager. VRAM budget, LRU eviction, keep-alive.
- **`src/arbiter/store.py`** — SQLite job persistence. Crash recovery.
- **`src/arbiter/worker.py`** — Thread pool for inference dispatch.
- **`src/arbiter/config.py`** — Pydantic config from `local/config.json`.
- **`src/arbiter/schemas.py`** — All API request/response schemas.
- **`src/arbiter/log.py`** — JSONL structured event logger.
- **`src/arbiter/cli.py`** — CLI that talks to the server over HTTP.
- **`src/arbiter/adapters/`** — One file per model. Each wraps load/unload/infer.
- **`src/arbiter/calibrate/`** — Measures VRAM, latency, concurrency per model.

## How to Run

```bash
./run server          # Start API server
./run test            # Fast tests (<30s, unit + integration)
./run test-all        # All tests except calibration
./run calibrate <model>  # Calibrate one model
./run ps              # Show loaded models
```

## How to Add a New Model

**IMPORTANT: Calibration is REQUIRED before any new model goes to production.**

### Step 1: Create the adapter

Create `src/arbiter/adapters/<model_name>.py`:

```python
from .base import ModelAdapter
from .registry import register

@register
class MyModelAdapter(ModelAdapter):
    model_id = "my-model"

    def __init__(self):
        self._model = None

    def load(self, device="cuda"):
        # Load model onto GPU
        self._model = ...

    def unload(self):
        del self._model
        self._model = None
        self._cleanup_gpu()

    def infer(self, params, output_dir, cancel_flag):
        self._check_cancel(cancel_flag)
        # Run inference, write result to output_dir
        # Return metadata dict: {"format": "png", "width": 1024, ...}
        ...

    def estimate_time(self, params):
        return 5000  # ms
```

### Step 2: Add to adapters/__init__.py

Add an import line so the adapter auto-registers:
```python
from . import my_model  # noqa: F401
```

### Step 3: Add job type to schemas.py

1. Add to `JobType` enum
2. Add to `JOB_TYPE_TO_MODEL` mapping
3. Create a params Pydantic model
4. Add to `JOB_TYPE_PARAMS` mapping

### Step 4: Add to config

Add the model entry to `local/config.default.json` with estimated values:
```json
"my-model": {
    "memory_gb": 4,
    "max_concurrent": 1,
    "keep_alive_seconds": 300,
    "avg_inference_ms": 5000,
    "load_ms": 10000,
    "auto_download": "org/model-name"
}
```

### Step 5: Run calibration (REQUIRED)

```bash
./run calibrate my-model
```

This produces `local/calibration/my-model.json` with measured values. Update `local/config.json` with the `config_entry` from the results.

### Step 6: Run tests

```bash
./run test
```

## Testing

- **Unit tests** (`tests/unit/`): No GPU needed. Test scheduler, memory manager, store, config.
- **Integration tests** (`tests/integration/`): FastAPI TestClient with mock adapters.
- **Calibration tests** (`tests/calibration/`): Manual only, per model.

All tests: `./run test` (excludes calibration and slow tests).

## Config

`local/config.json` (gitignored). Falls back to `local/config.default.json`.

Key per-model fields:
- `memory_gb` — VRAM when loaded (from calibration)
- `max_concurrent` — Max parallel inferences (from calibration)
- `avg_inference_ms` — Average inference time in ms (from calibration)
- `load_ms` — Model load time in ms (from calibration)
- `keep_alive_seconds` — Keep loaded after last use (default 300)
- `auto_download` — HuggingFace repo ID for auto-download
- Runtime registration/update goes through the Go control plane: `POST /v1/models` can add a new configured model live, and `PATCH /v1/models/{id}` with `reload_workers=true` replaces only that model's workers so other adapters keep serving.

## Logs

JSONL at `output/logs/arbiter-YYYY-MM-DD.jsonl`. Events: job lifecycle, model load/evict, memory snapshots.

## Reference Files

Arbiter supports "reference files" — binary files uploaded once and reused across multiple jobs without re-uploading.

### API

- `POST /v1/refs` — Upload a file (multipart `file` field or raw body with `?filename=`). Returns `{"ref_id": "abc123.wav"}`.
- `GET /v1/refs` — List all reference files.
- `GET /v1/refs/{id}` — Download a reference file.
- `DELETE /v1/refs/{id}` — Delete a reference file.

Files are stored in `output/refs/`.

### Usage in Jobs

Pass a `ref:` prefix in any `_file` parameter. This works with every adapter via `_resolve_media()`:

```json
{
  "type": "tts-clone",
  "params": {
    "text": "Hello",
    "ref_text": "Reference transcript",
    "ref_audio_file": "ref:abc123.wav"
  }
}
```

The `ref:` prefix is resolved in `_resolve_media()` in `src/arbiter/adapters/base.py`. No adapter-specific changes are needed.

## Key Design Decisions

- **Single process, ThreadPoolExecutor**: PyTorch releases GIL during CUDA ops. Threads share GPU memory efficiently.
- **SJF scheduling**: `priority = avg_inference_ms + (load_ms if not loaded else 0)`. Shortest jobs run first. Already-loaded models get natural priority.
- **SQLite queue**: Persistent, crash-recoverable. On restart, incomplete jobs are re-queued.
- **Dedup followers**: Duplicate requests are persisted as jobs with `state=following` and `error=following:<original_job_id>`. Startup recovery must requeue `scheduled`/`running` jobs, then reconcile followers so none remain attached to terminal or missing originals.
- **Async job API**: Submit → poll → get result. Server crashes are transparent to clients.
- **LRU eviction**: Models idle past keep_alive_seconds get evicted. When memory is needed, oldest idle model goes first.
- **Group adapters**: Sonic (8 sub-models) and LTX-2 (phased pipeline) load/unload atomically.

## Calibration Results (Grace Blackwell GB10, 128GB VRAM)

| Model | VRAM (GB) | Load Time | Inference Time | Max Concurrent |
|-------|-----------|-----------|----------------|----------------|
| birefnet | 0.83 | 5.4s | 1.0s | 2 |
| flux-schnell | 31.42 | 248s | 12s | 1 |
| moondream (v3) | 17.28 | 142s | 103s | 1 |
| whisper-large | 5.88 | 11.3s | 1.8s | 1 |
| tts-custom | 3.89 | 43s | 4s | 1 |
| tts-clone | 3.91 | 44s | ~4s | 1 |
| tts-design | ~3.9 | ~43s | ~5s | 1 |
| sonic (group) | 4.84 | 11s | ~45s | 1 |
| ltx2 (group) | ~55 | ~30s | ~120s | 1 |

Note: FLUX at 31GB was a surprise (estimated 12GB). Moondream3 at 17GB is 4x moondream2.
Sonic at 5GB is much lighter than the 15GB estimate.

## Known Issues & Compatibility

### transformers version
- **Must use transformers 4.57.3** — qwen-tts pins this exact version
- moondream3, FLUX, BiRefNet all work fine on 4.57.3
- transformers 5.x breaks qwen-tts import (`check_model_inputs` removed)

### Model-specific notes
- **Moondream3** (`moondream/moondream3-preview`): Upgraded from moondream2. Uses `dtype=` not `torch_dtype=` (deprecated). First inference triggers FlexAttention JIT compilation (~extra 30s). Consider torch.compile for production speed.
- **Whisper large-v3**: NOT thread-safe for concurrent calls. Errors at concurrency >= 2.
- **TTS output format**: `generate_custom_voice()` returns numpy arrays, not torch tensors. Adapter handles both via hasattr check.
- **TTS-clone**: Requires `ref_text` parameter alongside `ref_audio` for voice cloning.
- **Sonic**: Requires a real face in the input image — fails with "cannot access local variable 'bbox_s'" if no face detected.
- **LTX-2**: `load()` is instant (~2ms) — only creates config objects. Heavy model loading happens inside `infer()` per-phase. Memory manager should budget 55GB for the full job duration.
- **FLUX.1-schnell**: Needs `sentencepiece` package for tokenizer.
- **BiRefNet**: Needs `kornia` package.

## Key Dependencies

Core: fastapi, uvicorn, pydantic, httpx
ML: torch 2.10+cu130, transformers==4.57.3, diffusers, openai-whisper, qwen-tts
Model-specific: kornia (birefnet), sentencepiece (flux), omegaconf opencv-python-headless (sonic)
Packages: ltx-core, ltx-pipelines (installed from /home/darren/src/ltx2-spark/packages/)

## Model Weight Locations

Weights owned by Arbiter (in local/models/):
- `local/models/ltx2/` — LTX-2 checkpoints (moved from /home/darren/models/ltx2/)
- `local/models/sonic/` — Sonic checkpoints (moved from talking-head/Sonic/checkpoints/)
- Symlinks exist at the old locations pointing back here

HuggingFace cache (loaded by repo ID, shared ~/.cache/huggingface/):
- FLUX.1-schnell, BiRefNet, Moondream3, Qwen3-TTS variants

Whisper cache: ~/.cache/whisper/large-v3.pt

Note: tts-design (Qwen3-TTS-12Hz-1.7B-VoiceDesign) is NOT downloaded yet.

## Running as a Daemon

Arbiter runs under `auto` (process manager):

```bash
# It's already registered:
auto ps                    # Check status
auto start arbiter         # Start
auto stop arbiter          # Stop
auto restart arbiter       # Restart
auto log arbiter           # View logs
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/darrenoakey)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/darrenoakey)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
