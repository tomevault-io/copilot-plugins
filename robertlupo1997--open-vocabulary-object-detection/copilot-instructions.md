## open-vocabulary-object-detection

> Production-ready detection system combining **Grounding DINO** (text-conditioned detection) and **SAM 2** (segmentation masks). Query images with natural language to detect arbitrary object categories.

# OVOD - Open-Vocabulary Object Detection

Production-ready detection system combining **Grounding DINO** (text-conditioned detection) and **SAM 2** (segmentation masks). Query images with natural language to detect arbitrary object categories.

## Quick Reference

```bash
# Environment setup
conda env create -f env-ovod.yml && conda activate ovod
make quickstart                    # Full setup with data linking

# Development
export PYTHONPATH=$(pwd)/repo      # Required for imports
make test                          # Run basic tests
make lint                          # Python syntax check
make clean                         # Remove __pycache__, *.pyc

# Demo & Evaluation
make demo                          # Streamlit UI on :8501
make eval-50                       # 50 images, common prompt
make eval-200                      # 200 images evaluation
make eval-full                     # Full 5000 image COCO eval
```

## Tech Stack

- **Python**: 3.10
- **ML**: PyTorch 2.5.1 (CUDA 12.1), Transformers 4.55, Timm 1.0
- **Models**: Grounding DINO (SwinT-OGC), SAM 2 (Hiera variants)
- **Evaluation**: pycocotools 2.0
- **UI**: Streamlit 1.48
- **Env**: Conda (`env-ovod.yml`), locked deps (`requirements.lock.txt`)

## Project Structure

```
repo/
├── ovod/
│   ├── pipeline.py      # Main OVODPipeline orchestrator
│   └── config.py        # Paths, model URLs, thresholds
├── src/
│   ├── detector.py      # GroundingDINODetector wrapper
│   ├── segmenter.py     # SAM2Segmenter wrapper
│   ├── nms.py           # NMS utilities, box IoU
│   ├── prompts.py       # PromptProcessor for text parsing
│   └── visualize.py     # Box drawing, color generation
├── metrics/
│   ├── coco_eval.py     # COCO evaluation
│   └── benchmark.py     # Latency benchmarking
├── demo_app.py          # Streamlit web interface
├── eval.py              # COCO eval with prompt strategies
└── Makefile             # Production targets
tests/                   # Pytest suite with mocked components
notebooks/               # OVOD_Explainer_TLDR.ipynb
```

## Key Entry Points

| File | Purpose |
|------|---------|
| `repo/ovod/pipeline.py` | Core pipeline - start here |
| `repo/demo_app.py` | Interactive Streamlit demo |
| `repo/eval.py` | COCO evaluation script |
| `repo/src/detector.py` | Grounding DINO integration |
| `repo/src/segmenter.py` | SAM 2 integration |

## Code Patterns

**Device handling:**
```python
device = "cuda" if torch.cuda.is_available() else "cpu"
pipeline = OVODPipeline(device=device)
```

**Grounding DINO prompt format:**
```python
# Must end with periods separated by spaces
"person . car . dog ."
```

**Result dictionary structure:**
```python
result = {
    "boxes": np.ndarray,      # (N, 4) xyxy format
    "labels": List[str],      # N label names
    "scores": np.ndarray,     # (N,) confidence
    "masks": List[np.ndarray], # N binary masks
    "timings": dict           # Latency breakdown
}
```

**CI compatibility:**
```python
if os.getenv("OVOD_SKIP_SAM2") == "1":
    SAM2Segmenter = None  # Skip heavy imports in CI
```

## Testing

```bash
pytest tests/ -v                   # Full suite
pytest tests/test_pipeline.py      # Specific file
pytest -m "not slow"               # Skip slow tests (default)
```

- Tests use mocked detector/segmenter for CI
- Markers: `slow`, `integration`, `unit`
- Fixtures in `tests/conftest.py`

## Code Style

- 4-space indentation (Python), 2-space (YAML)
- Type hints on function signatures
- Docstrings with Args/Returns sections
- Error handling with graceful degradation

## Prompt Strategies

| Strategy | Classes | Use Case |
|----------|---------|----------|
| `person` | 1 | Person detection only |
| `common` | 18 | General objects |
| `full` | 80 | Complete COCO classes |

## Performance

~265ms/image on RTX 3070 (detection ~100ms, segmentation ~45ms, NMS ~5ms)

## Common Issues

- **Import errors**: Ensure `PYTHONPATH=$(pwd)/repo`
- **SAM2 not found**: Check `env-ovod.yml` installed correctly
- **CI failures**: Set `OVOD_SKIP_SAM2=1` for lightweight tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robertlupo1997) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
