## dmriprep

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

dMRIPrep is a robust, analysis-agnostic preprocessing pipeline for diffusion MRI (dMRI) data. It's part of the NiPreps (Neuroimaging Preprocessing) ecosystem and automatically adapts best-practice workflows to diverse dMRI datasets.

- **Documentation**: https://www.nipreps.org/dmriprep
- **Docker Hub**: https://hub.docker.com/r/nipreps/dmriprep
- **Python**: 3.10, 3.11, 3.12, 3.13

## Common Commands

### Development Setup
```bash
# Install pixi (if not already installed)
curl -fsSL https://pixi.sh/install.sh | bash

# Create development environment (Linux only, see note below)
pixi install

# Or specific environment
pixi install -e test
```

**Note:** Pixi environments currently only support `linux-64` due to a dependency
conflict between nifreeze and nitransforms. On non-`linux-64` platforms, the
recommended development path is micromamba with a `dmriprep` environment:
```bash
micromamba activate dmriprep
python -m pip install -e ".[dev,test]"
```
If micromamba is unavailable, fall back to:
```bash
pip install -e ".[dev,test]"
```

### Testing
```bash
# Run tests
pixi run test

# Run tests with coverage
pixi run test-cov

# Run tests via tox (all Python versions)
pixi run -e test tox

# Run specific test file (within pixi shell)
pixi shell -e test
pytest dmriprep/cli/tests/test_parser.py -v
```
With micromamba:
```bash
micromamba run -n dmriprep python -m pytest -q
```

Documentation should build, as it builds workflows and serves for unit test:

```bash
# When pixi is available:
SPHINXOPTS="-W" pixi run -e docs build-docs
```

With micromamba:
```bash
SPHINXOPTS="-W" micromamba run -n dmriprep make -C docs/ html
```

### Code Quality
```bash
# Check style
pixi run -e dev lint
pixi run -e dev style

# Auto-fix style issues
pixi run -e dev style-fix
pixi run -e dev format

# Spellcheck (via tox)
pixi run -e test tox -e spellcheck
```

### Building
```bash
# Install in development mode (alternative to pixi)
pip install -e ".[dev,test]"

# Build distributions
python -m build
```

### Documentation
```bash
# Build docs
pixi run -e docs build-docs

# Live preview (serves at localhost:8000)
pixi run -e docs docs
```

## Architecture

### Core Components

- **`dmriprep/cli/`** - Command-line interface
  - `run.py` - Main entry point (`dmriprep` command)
  - `parser.py` - Argument parsing
  - `workflow.py` - CLI workflow orchestration

- **`dmriprep/config/`** - Runtime configuration system (ToML-based, cross-process communication via filesystem)

- **`dmriprep/workflows/`** - Nipype workflow definitions
  - `base.py` - Main dMRIPrep workflow
  - `dwi/base.py` - DWI preprocessing pipeline
  - `dwi/eddy.py` - Eddy current correction

- **`dmriprep/interfaces/`** - Custom Nipype interfaces
  - `bids.py` - BIDS-specific interfaces
  - `images.py` - Image processing
  - `vectors.py` - Gradient/vector handling
  - `reports.py` - Report generation

- **`dmriprep/utils/`** - Utility functions

### Key Dependencies

- **Nipype** - Workflow engine
- **NiWorkflows/SMRIPrep/SDCFlows** - Shared NiPreps components
- **DIPY** - Diffusion imaging processing
- **PyBIDS** - BIDS dataset handling

## Code Style

- Line length: 99 characters
- Quotes: Single quotes
- Linter: Ruff (replaces flake8/black/isort)
- Docstrings: NumPy style

## Testing Notes

- Tests are located within the `dmriprep/` package (not a separate `tests/` directory)
- Some tests require DIPY test data, cached in `~/.cache/data/` or `$DMRIPREP_TESTS_DATA`
- TemplateFlow templates cached in `~/.cache/templateflow/`
- `conftest.py` provides `dipy_test_data` fixture for Sherbrooke HARDI dataset

## Configuration System

The `dmriprep/config/` module provides singleton configuration management:
- ToML-based storage
- Cross-process communication via filesystem
- Sections: `environment`, `execution`, `workflow`, `nipype`, `loggers`

---
> Source: [nipreps/dmriprep](https://github.com/nipreps/dmriprep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
