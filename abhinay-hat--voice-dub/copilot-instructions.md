## voice-dub

> Manages sequential loading/unloading of ML models to maximize VRAM efficiency. Always use this when working with models:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Voice Dub is a local video dubbing application that transforms videos in any language into English-dubbed versions with voice cloning, emotion preservation, and lip synchronization. The system runs entirely on local hardware using open-source AI models (NVIDIA RTX 5090 with 32GB VRAM).

**Current Status**: Phase 1 complete (Environment & Foundation). The project follows a structured phase-based development approach managed through the GSD (Get Stuff Done) workflow in `.planning/`.

## Hardware & Environment Requirements

### Critical GPU Setup
- **GPU**: NVIDIA RTX 5090 (32GB VRAM) with compute capability 12.0 (sm_120)
- **CUDA**: Version 12.8 or higher
- **PyTorch**: Nightly builds REQUIRED (stable releases don't support sm_120)

### Environment Validation
Always validate the GPU environment before development work:

```bash
python src/utils/gpu_validation.py
```

This script checks CUDA availability, GPU model, compute capability, VRAM, and prevents silent CPU fallback. The environment is configured with critical CUDA allocator settings to prevent memory fragmentation:
- `expandable_segments:True` - Prevents memory fragmentation
- `max_split_size_mb:128` - Avoids splitting large blocks
- `garbage_collection_threshold:0.8` - Reclaims memory at 80% usage

These settings are automatically applied when importing `src.utils.gpu_validation`.

## Development Commands

### Setup
```bash
# Create and activate virtual environment
python -m venv venv
.\venv\Scripts\activate  # Windows
source venv/bin/activate # Linux/macOS

# Install dependencies (includes PyTorch nightly with CUDA 12.8)
pip install --upgrade pip
pip install -r requirements.txt
```

### Testing
```bash
# Run complete GPU environment test suite
python tests/test_gpu_environment.py

# Validate GPU setup only
python src/utils/gpu_validation.py
```

### Python Path
When running scripts directly, add project root to Python path:
```python
from pathlib import Path
import sys
project_root = Path(__file__).parent.parent
sys.path.insert(0, str(project_root))
```

## Architecture

### Core Design Principles

**Sequential Model Loading**: The system uses a `ModelManager` to load only one ML model at a time, preventing VRAM exhaustion. When loading a new model, the manager automatically unloads the previous one.

**Pipeline Architecture**: The dubbing process is organized into sequential stages:
1. Video upload and extraction (FFmpeg)
2. Speech recognition (Whisper Large V3)
3. Translation (SeamlessM4T v2)
4. Voice cloning (XTTS-v2)
5. Lip synchronization (Wav2Lip/LatentSync)
6. Assembly and export

### Key Components

#### ModelManager (src/models/model_manager.py)
Manages sequential loading/unloading of ML models to maximize VRAM efficiency. Always use this when working with models:

```python
from src.models.model_manager import ModelManager

manager = ModelManager()

# Load Whisper for speech recognition
whisper = manager.load_model("whisper", lambda: load_whisper())
transcripts = whisper.transcribe(audio)

# Automatically unloads Whisper before loading SeamlessM4T
seamless = manager.load_model("seamless", lambda: load_seamless())
translations = seamless.translate(transcripts)
```

The manager handles:
- Moving models to CPU before deletion
- Forcing garbage collection
- Clearing CUDA cache
- Preventing memory leaks from circular references

#### GPU Validation (src/utils/gpu_validation.py)
Critical utility that prevents silent CPU fallback. Must be imported before any GPU operations to set environment variables. Validates:
- CUDA availability
- RTX 5090 detection
- Compute capability sm_120
- VRAM accessibility (32GB)
- Actual GPU allocation (no CPU fallback)

#### Memory Monitor (src/utils/memory_monitor.py)
Provides real-time VRAM tracking during development:

```python
from src.utils.memory_monitor import print_gpu_memory_summary

print_gpu_memory_summary("Before model load: ")
model = load_large_model()
print_gpu_memory_summary("After model load: ")
```

Returns:
- `allocated`: Actually in use by tensors
- `reserved`: Allocated by PyTorch (includes cache)
- `free`: Available for PyTorch to reserve
- `total`: Total GPU memory

#### Configuration (src/config/settings.py)
Centralized constants for paths, model configurations, and pipeline parameters:
- Directory paths (data, models, temp, outputs)
- GPU settings (device index, compute capability)
- Model identifiers (Whisper, SeamlessM4T, XTTS, Wav2Lip)
- Pipeline constraints (max duration, supported formats)

### Project Structure

```
src/
├── pipeline/          # Stage orchestration (future)
├── stages/            # Individual processing stages (future)
├── models/            # Model loading/unloading logic
│   ├── model_manager.py    # Sequential model manager
│   └── __init__.py
├── utils/             # GPU validation, memory monitoring
│   ├── gpu_validation.py   # RTX 5090 environment checks
│   ├── memory_monitor.py   # VRAM tracking utilities
│   └── __init__.py
└── config/            # Configuration constants
    ├── settings.py         # Paths, models, GPU settings
    └── __init__.py

data/
├── raw/               # Original uploaded videos
├── temp/              # Intermediate files
└── outputs/           # Final processed videos

models/                # Downloaded model weights cache
tests/                 # Test files
.planning/             # GSD workflow (roadmap, plans, research)
```

## Development Workflow

### GSD (Get Stuff Done) Integration
This project uses the GSD workflow system. Key files in `.planning/`:
- `PROJECT.md`: Core value, requirements, constraints
- `ROADMAP.md`: Phase breakdown with success criteria
- `STATE.md`: Current execution state
- `phases/`: Per-phase planning, research, and execution docs

When working on a phase:
1. Read phase context in `.planning/phases/{phase-name}/`
2. Follow plans in `{phase-number}-{plan-number}-PLAN.md`
3. Update execution summaries as work progresses

### Memory Management Best Practices

**Always use ModelManager for ML models**. Manual model management risks memory leaks.

**Monitor VRAM during development** using `print_gpu_memory_summary()` to catch leaks early.

**Test allocation before inference** to prevent silent CPU fallback:
```python
# Quick CUDA allocation test
test = torch.randn(1000, device='cuda')
assert test.is_cuda, "Tensor not on GPU!"
del test
torch.cuda.empty_cache()
```

**Clear cache after model unload**:
```python
del model
gc.collect()
torch.cuda.empty_cache()
```

### Common Issues

**"no kernel image is available for execution"**: PyTorch stable release installed instead of nightly. Install PyTorch nightly with CUDA 12.8:
```bash
pip uninstall torch torchvision torchaudio
pip install --pre torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/nightly/cu128
```

**"CUDA is not available"**: Either NVIDIA drivers missing or PyTorch CPU-only version installed. Run `nvidia-smi` to verify drivers, then reinstall PyTorch nightly.

**Gradual memory leak**: Model not properly unloaded. Always use `ModelManager` instead of manual model management. If implementing custom cleanup, follow the 4-step process: move to CPU, delete references, garbage collect, clear CUDA cache.

## Model Pipeline (Planned)

The following models will be integrated in future phases:

- **Whisper Large V3**: Speech-to-text (20-30 languages, speaker diarization)
- **SeamlessM4T v2**: Context-aware translation with vocal expression preservation
- **XTTS-v2**: Voice cloning with emotion preservation (non-commercial license)
- **Wav2Lip/LatentSync**: Lip synchronization to English audio
- **FFmpeg**: Video/audio extraction and assembly

Each model loads sequentially through ModelManager to fit in 32GB VRAM.

## Testing Approach

All GPU-related functionality must be tested against RTX 5090 hardware. The test suite in `tests/test_gpu_environment.py` validates:
1. GPU validation utility correctness
2. Memory monitoring accuracy
3. ModelManager sequential loading
4. CUDA environment variables

Run tests after any changes to GPU management code:
```bash
python tests/test_gpu_environment.py
```

Expected output: "ALL TESTS PASSED ✓"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abhinay-hat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
