## aion

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AION (AstronomIcal Omnimodal Network) is a large omnimodal transformer model for astronomical surveys. It processes 39 distinct astronomical data modalities using a two-stage architecture:

1. **Modality-specific tokenizers** transform raw inputs (images, spectra, catalogs, scalars) into discrete tokens
2. **Unified encoder-decoder transformer** processes all token streams via multimodal masked modeling (4M)

The model comes in three variants: Base (300M), Large (800M), and XLarge (3B parameters).

## Development Commands

### Testing
```bash
pytest                           # Run all tests
pytest tests/codecs/            # Run codec tests only
pytest tests/test_data/         # Uses pre-computed test data for validation
```

### Linting and Code Quality
```bash
ruff check .                    # Check code style and lint
ruff check . --fix             # Auto-fix linting issues
```

### Installation for Development
```bash
pip install -e .[torch,dev]    # Install in editable mode with dev dependencies
```

### Documentation
```bash
cd docs && make html           # Build Sphinx documentation
```

## Architecture Overview

### Core Components

- **`aion/model.py`**: Main AION wrapper class, inherits from FM (4M) transformer
- **`aion/fourm/`**: 4M (Four-Modal) transformer implementation
  - `fm.py`: Core transformer architecture with encoder-decoder blocks
  - `modality_info.py`: Configuration for all 39 supported modalities
  - `encoder_embeddings.py` / `decoder_embeddings.py`: Modality-specific embedding layers
- **`aion/codecs/`**: Modality tokenization system
  - `manager.py`: Dynamic codec loading and management
  - `base.py`: Abstract base codec class
  - Individual codec implementations for images, spectra, scalars, etc.
- **`aion/modalities.py`**: Type definitions for all astronomical data types

### Key Design Patterns

1. **Modality System**: Each astronomical data type (flux, spectrum, catalog) has:
   - A modality class in `modalities.py` defining data structure
   - A codec in `codecs/` for tokenization
   - Embedding layers in `fourm/` for the transformer

2. **Token Keys**: Each modality has a `token_key` (e.g., `tok_image`, `tok_spectrum_sdss`) that maps between modalities and model components

3. **HuggingFace Integration**: Models and codecs are distributed via HuggingFace Hub with `from_pretrained()` methods

## Code Conventions

- Type hints are mandatory, using `jaxtyping` for tensor shapes (e.g., `Float[Tensor, "batch height width"]`)
- Modality classes use `@dataclass` and inherit from `BaseModality`
- All tensor operations should handle device placement explicitly
- Test data is pre-computed and stored in `tests/test_data/` as `.pt` files

## Testing Strategy

Tests validate both encoding and decoding for each modality using pre-computed reference data. The test pattern is:
1. Load input, encoded, and decoded reference tensors
2. Run codec encode/decode operations
3. Assert outputs match reference data within tolerance

Test files follow naming: `test_{modality}_codec.py`

## Astronomical Context

The model processes data from major surveys:
- **Legacy Survey**: Optical images and catalogs (g,r,i,z bands + WISE)
- **HSC (Hyper Suprime-Cam)**: Deep optical imaging (g,r,i,z,y bands)
- **Gaia**: Astrometry, photometry, and BP/RP spectra
- **SDSS/DESI**: Optical spectra

Each modality represents different physical measurements (flux, shape parameters, coordinates, extinction, etc.) that the model learns to correlate.

---
> Source: [PolymathicAI/AION](https://github.com/PolymathicAI/AION) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
