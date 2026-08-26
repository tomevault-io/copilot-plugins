## mask2former

> This is a PyTorch reimplementation of Mask2Former for image segmentation, built around **timm backbones** and transformer decoder architecture. The project is in **alpha status** with core functionality under active development.

# Copilot Instructions for Mask2Former

## Project Overview

This is a PyTorch reimplementation of Mask2Former for image segmentation, built around **timm backbones** and transformer decoder architecture. The project is in **alpha status** with core functionality under active development.

## Architecture Patterns

### Core Components (`src/mask2former/modeling/`)

- **`backbone.py`**: Thin wrapper around timm models for multi-scale feature extraction
  - Uses `timm.create_model(features_only=True)` with configurable `out_indices`
  - Standard pattern: `(1, 2, 3)` for ConvNeXt, `(2, 3, 4)` for ResNet
- **`decoder.py`**: Transformer decoder with custom attention mechanisms
  - `CrossAttention`: Queries attend to image features through boolean masks
  - `SelfAttention`: Standard transformer self-attention
  - Uses `torch.nn.functional.scaled_dot_product_attention` (PyTorch 2.0+)
- **`pe.py`**: 2D sinusoidal positional embeddings for spatial understanding
- **`common/`**: Shared components (FFN, MLP) following transformer conventions

### Data Flow Pattern

1. Images → timm backbone → multi-scale features
2. Features → positional embeddings → transformer decoder
3. Query features → mask embeddings → segmentation masks

## Development Workflow

### Essential Commands (via Makefile)

```bash
make check          # Complete quality pipeline: lint + format-check + type-check + test
make test-cov       # Tests with HTML coverage report
make docs           # Local docs server with auto-reload
make clean          # Remove all build artifacts and caches
```

### Testing Conventions

- Tests use **pytest** with **PIL/timm** for realistic image processing
- Standard fixture: `sample_image` from HuggingFace documentation images
- Test pattern: Input validation → Model forward pass → Output shape assertions
- Coverage target: HTML reports in `htmlcov/`

### Code Quality Stack

- **ruff**: Linting and formatting (replaces black/flake8/isort)
- **pyright**: Type checking (strict mode)
- **pre-commit**: Automated hooks for quality gates
- **uv**: Fast Python package/environment manager

## Project-Specific Conventions

### Logging

Always prefer using python's built-in `logging` module over print statements for debug/info output.

```python
import logging
logger = logging.getLogger(__name__)
logger.info("Informational message")
```

### Tensor Shape Documentation

Always document tensor shapes in docstrings using the pattern:

```python
Args:
    x (torch.Tensor): Input tensor.
        Shape: (batch_size, channels, height, width)
```

### Attention Module Pattern

- Separate query/key/value projectors as `nn.Linear` layers
- Multi-head reshaping: `view(batch, seq, heads, head_dim).transpose(1, 2)`
- Use PyTorch's `scaled_dot_product_attention` for efficiency
- Boolean masks for attention control (False = ignore)

### Model Configuration

- Default backbone: `"convnext_base.dinov3_lvd1689m"` from timm
- Embedding dimensions must be divisible by number of heads
- Positional embedding dimension must be divisible by 4

### Dependencies & Compatibility

- **Python 3.11+** required
- **PyTorch 2.9+** for latest attention functions
- **timm 1.0.20+** for modern backbone support
- Use `uv sync --all-groups` for dependency management

## Key Files for Context

- `pyproject.toml`: Complete project configuration, dependencies, and tool settings
- `Makefile`: All development commands and workflows
- `tests/conftest.py`: Shared test fixtures and configuration
- `src/mask2former/modeling/decoder.py`: Core transformer implementation (485 lines)

## Integration Points

- **timm ecosystem**: Backbone feature extraction with standard transforms
- **PyTorch Lightning**: Ready for training loop integration (alpha status)
- **HuggingFace**: Compatible tensor formats for model hub integration

---
> Source: [NoeFontana/mask2former](https://github.com/NoeFontana/mask2former) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
