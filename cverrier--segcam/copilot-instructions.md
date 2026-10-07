## segcam

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SegCam is a real-time semantic segmentation application that processes video from an iPhone (via Continuity Camera) on a MacBook M1, using YOLOv8-seg for instance segmentation with contour visualization.

## Development Commands

```bash
# Install dependencies
uv sync

# Run tests
uv run pytest

# Run tests (quiet mode, concise failures)
uv run pytest -q --tb=short

# Run a single test file
uv run pytest tests/test_config.py -v

# Run the application
uv run segcam
uv run segcam --config path/to/config.toml
uv run segcam --backend pytorch
```

## Architecture

The project follows a modular architecture with clear separation between:

- **config.py** - TOML configuration loading with validation via dataclasses
- **logging.py** - Structured logging with JSON output and debug mode support
- **capture.py** - Video capture with camera enumeration, interactive selection, and connection resilience
- **inference/** - Pluggable backends (tinygrad primary, PyTorch fallback)
  - `base.py` - Abstract `InferenceBackend` interface with `Detection` dataclass
  - `tinygrad_backend.py` - ONNX execution via tinygrad's Metal backend
  - `pytorch_backend.py` - ultralytics YOLO with MPS acceleration
- **visualization.py** - Contour rendering and debug overlay
- **app.py** - Main application loop wiring all components
- **__main__.py** - CLI entry point

### Inference Flow

1. Frame captured from Continuity Camera at 720p @ 30fps
2. Backend runs YOLOv8-seg inference
3. Post-processing: confidence filtering, NMS, mask assembly
4. Contours extraction
5. Visualization draws contours with HSV-distributed colors per class

### Key Interfaces

```python
# Detection result from inference
@dataclass
class Detection:
    class_id: int
    class_name: str
    confidence: float
    bbox: tuple[int, int, int, int]  # x1, y1, x2, y2
    mask: np.ndarray  # Binary mask
    contours: list[np.ndarray]

# All backends implement this
class InferenceBackend(ABC):
    def load(self) -> None: ...
    def predict(self, frame: np.ndarray) -> list[Detection]: ...
    def class_names(self) -> list[str]: ...
```

## Code Style

- Python 3.13+, managed with `uv`
- Ruff for linting/formatting (line length 88, indent width 2)
- Use indent width 2 in docstrings
- Type hints required
- Use `numpy.typing.NDArray` for NumPy array type hints (e.g., `NDArray[np.uint8]`)
- Tests use pytest with `--import-mode=importlib`
- Use comments sparingly; only when mentioning important points not obvious from the code
- Use f-strings for string formatting; avoid `.format()` syntax (except in logging calls)
- Do not add docstrings when the function signature is self-explanatory. Same for test functions.
- Do not use `is True` or `is False`; use direct boolean checks instead.

---
> Source: [cverrier/segcam](https://github.com/cverrier/segcam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
