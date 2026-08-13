## ml-research-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **Copier template** for generating ML research projects with PyTorch Lightning + Hydra + W&B. It's NOT a Python project itself—it's a template that generates Python projects through Jinja2 templating.

**Key distinction**: Files ending in `.jinja` are templates that get rendered when users run `copier copy`. Do not run Python commands directly in this repository.

## Development Workflow

### Testing Template Generation

The primary development workflow is testing template generation:

```bash
# Test with defaults
uvx copier copy --trust --defaults . /tmp/test-project

# Test with specific configuration
uvx copier copy --trust \
  --data project_name="test-ml" \
  --data python_version="3.12" \
  --data package_manager="uv" \
  --data pytorch_cuda_preset="pytorch-2.8.0-cuda-12.6" \
  --data use_lightning=true \
  --data use_hydra=true \
  --data logger_choice="tensorboard" \
  . /tmp/test-project
```

### Verifying Generated Projects

After generating a test project, verify it works:

```bash
cd /tmp/test-project

# With uv (default)
uv sync
uv run pytest tests/ -v
uv run ruff check .
uv run ruff format --check .
uv run ty check src/
uv run python src/<package_name>/train.py trainer.max_epochs=1

# With pixi
pixi install
pixi run pytest tests/
pixi run ruff check .
```

### Pre-commit Hooks

```bash
# Install (one-time)
pip install pre-commit
pre-commit install

# Run manually
pre-commit run --all-files
```

## Unified pyproject.toml Approach

**Key Change**: Since version 0.2.0, this template uses a **unified pyproject.toml** for both UV and Pixi package managers. Previously, Pixi users had separate `pixi.toml` and `pyproject.toml` files.

**Benefits**:
- Single source of truth for project configuration
- No duplicate dependency management
- Easier migration between package managers
- Follows Pixi 0.30+ `pyproject.toml` support

**Package Manager Differences**:

| Aspect | UV | Pixi |
|--------|-----|------|
| PyTorch source | PyPI wheel index (bundled CUDA) | PyPI + conda CUDA toolkit |
| CUDA isolation | Bundled in wheel | Isolated in `.pixi/envs/` |
| Config section | `[tool.uv]`, `[dependency-groups]` | `[tool.pixi.*]` |
| Lock file | `uv.lock` | `pixi.lock` |

## Template Architecture

### Configuration Flow (copier.yml)

The `copier.yml` file defines:
1. **User questions**: Interactive prompts for project configuration
2. **Conditional logic**: Some questions only appear based on previous answers
3. **Preset mappings**: PyTorch/CUDA version combinations (9 presets)
4. **Post-generation tasks**: Automated setup (git init, dependency installation, quality checks)

Key configuration options:
- **Package managers**: uv (fast, pip-compatible) or pixi (conda-based, GPU auto-setup)
- **PyTorch presets**: 9 combinations from PyTorch 2.4.1-2.9.0 with CUDA 11.8-13.0
- **ML frameworks**: PyTorch Lightning (2.2-2.4), Hydra (1.2-1.3)
- **Experiment tracking**: TensorBoard, W&B, MLflow, both, or none

### Template Structure

```
.
├── copier.yml                    # Template configuration and questions
├── pyproject.toml.jinja          # Unified config for BOTH uv and pixi
├── ruff.toml.jinja               # Linter/formatter config
├── src/{{ package_name }}/
│   ├── train.py.jinja            # Hydra-based training script
│   ├── models/
│   │   └── base_model.py.jinja   # LightningModule with optimizer config
│   └── data/
│       └── datamodule.py.jinja   # LightningDataModule (MNIST example)
├── configs/                      # Hydra configuration hierarchy
│   ├── config.yaml.jinja         # Main config with defaults
│   ├── model/default.yaml.jinja
│   ├── data/default.yaml.jinja
│   ├── trainer/default.yaml.jinja
│   ├── logger/*.yaml.jinja       # TensorBoard/W&B/MLflow configs
│   └── experiment/baseline.yaml.jinja
└── tests/
    └── test_{{ package_name }}.py.jinja
```

### Generated Project Architecture

When users generate a project, they get:

**Training flow**:
1. `train.py` loads Hydra config from `configs/config.yaml`
2. Hydra instantiates DataModule, Model, Trainer, Logger from configs
3. PyTorch Lightning handles training loop, checkpointing, logging
4. Experiment tracking goes to TensorBoard/W&B/MLflow

**Hydra configuration composition**:
- Base configs in `configs/{model,data,trainer,logger}/`
- Override via CLI: `python train.py trainer.max_epochs=100`
- Experiment configs in `configs/experiment/` combine multiple overrides
- Multi-run sweeps: `python train.py -m data.batch_size=32,64,128`

**Package management**:
- `uv`: Uses `[tool.uv]` sections in `pyproject.toml` with PyTorch wheel index
- `pixi`: Uses `[tool.pixi.*]` sections in `pyproject.toml` with conda CUDA + PyPI PyTorch

## Common Development Tasks

### Adding New PyTorch/CUDA Preset

1. Add choice to `copier.yml` under `pytorch_cuda_preset`
2. Add version extraction logic in `pyproject.toml.jinja` (lines 9-36)
3. Update both UV and Pixi sections to use new versions
4. Update README.md preset table
5. Test generation with BOTH uv and pixi

### Modifying Generated Code Templates

1. Edit `.jinja` files in `src/{{ package_name }}/`
2. Use Jinja2 conditionals for optional features:
   ```jinja
   {% if use_lightning -%}
   import pytorch_lightning as pl
   {% endif -%}
   ```
3. Test with feature enabled and disabled
4. Verify linting passes in generated project

### Adding New Configuration Options

1. Add question to `copier.yml`:
   ```yaml
   my_feature:
     type: bool
     help: "Enable my feature?"
     default: false
   ```
2. Use in templates: `{% if my_feature %}...{% endif %}`
3. Add to `_exclude` in `copier.yml` if files should be conditionally excluded
4. Update README.md and CONTRIBUTING.md

## Testing Requirements

Before submitting changes, verify:

- [ ] Template generates with `--defaults`
- [ ] Template generates with uv and pixi
- [ ] Template generates with different Python versions (3.10-3.13)
- [ ] Template generates with different PyTorch presets
- [ ] Generated project: `uv sync` / `pixi install` succeeds
- [ ] Generated project: `pytest` passes
- [ ] Generated project: `ruff check` passes
- [ ] Generated project: `ty check` passes
- [ ] Generated project: Training runs without error
- [ ] Pre-commit hooks pass
- [ ] copier.yml has valid YAML syntax
- [ ] All .jinja files have valid Jinja2 syntax

## Jinja2 Template Syntax

Key patterns used in this template:

```jinja
{# Comment - not rendered #}

{% if condition -%}      # Conditional (- strips whitespace)
content
{% endif -%}

{{ variable }}           # Variable substitution

{% for item in list %}   # Loops
{{ item }}
{% endfor %}

{{ var | replace('-', '_') }}  # Filters
```

## CI/CD

GitHub Actions test template generation across:
- OS: Ubuntu, macOS
- Python: 3.10, 3.11, 3.12, 3.13
- Package managers: uv, pixi
- PyTorch presets: Multiple combinations

See `.github/workflows/test-template.yml` for the test matrix.

## Key Files

- `copier.yml`: Template configuration, questions, post-generation tasks
- `pyproject.toml.jinja`: **Unified template** with conditional `{% if package_manager == 'uv' %}` / `{% if package_manager == 'pixi' %}` blocks
- `src/{{ package_name }}/train.py.jinja`: Hydra training script with Lightning integration
- `src/{{ package_name }}/models/base_model.py.jinja`: LightningModule with Hydra optimizer/scheduler config
- `configs/config.yaml.jinja`: Hydra composition root with defaults list

## CUDA Isolation (Pixi)

Pixi projects use **conda CUDA toolkit** isolated in `.pixi/envs/`:

```toml
[tool.pixi.system-requirements]
cuda = "12.6"

[tool.pixi.activation]
env = { PYTHONNOUSERSITE = "1" }  # CRITICAL: Isolate from ~/.local

[tool.pixi.dependencies]
python = "3.11.*"
cuda = { version = "12.6.*", channel = "nvidia" }
cudnn = { version = "*", channel = "nvidia" }

[tool.pixi.pypi-dependencies]
torch = "==2.8.0"  # PyPI PyTorch uses conda CUDA
```

**CRITICAL**: `PYTHONNOUSERSITE=1` prevents Python from loading packages from `~/.local/lib/pythonX.Y/site-packages/`, ensuring complete isolation from host packages.

**Verification**:
```bash
pixi run python -c "import os; print(os.environ.get('PYTHONNOUSERSITE'))"  # Should print: 1
pixi run python -c "import torch; print(torch.__file__)"  # Should be in .pixi/envs/
pixi list | grep cuda  # Shows isolated CUDA packages
ls .pixi/envs/default/lib/libcuda*  # CUDA libs in pixi env
```

## Critical Implementation Details

### PYTHONNOUSERSITE Environment Variable

**Problem**: By default, Python includes user site-packages (`~/.local/lib/pythonX.Y/site-packages/`) in `sys.path`, which can interfere with Pixi's isolated environment.

**Solution**: Set `PYTHONNOUSERSITE=1` in Pixi activation:

```toml
[tool.pixi.activation]
env = { PYTHONNOUSERSITE = "1" }
```

**Reference**: [Conflicts with per user site-packages directory · Issue #2859 · prefix-dev/pixi](https://github.com/prefix-dev/pixi/issues/2859)

### PyTorch Source Strategy

**Important Discovery**: PyTorch 2.5.0+ is no longer available via conda channels.

**Current approach**:
- **UV**: PyTorch from PyPI wheel index (CUDA bundled in wheel)
- **Pixi**: PyTorch from PyPI + CUDA toolkit from conda (isolated)

```toml
# Pixi configuration
[tool.pixi.dependencies]
cuda = { version = "12.6.*", channel = "nvidia" }  # Isolated CUDA

[tool.pixi.pypi-dependencies]
torch = "==2.8.0"  # PyPI PyTorch uses conda CUDA
```

## Pitfalls to Avoid

1. **Don't run Python in template repo**: This isn't a Python project—test generated projects instead
2. **Maintain Jinja2 syntax**: Use `{%-` and `-%}` to control whitespace in generated files
3. **Test both package managers**: uv and pixi have different dependency resolution
   - UV: PyTorch from wheel index (bundled CUDA)
   - Pixi: PyTorch from PyPI + conda CUDA toolkit (isolated)
4. **Version compatibility**: Verify PyTorch/Lightning/Hydra version combinations work
5. **Update both sections**: When changing dependencies, update BOTH `{% if package_manager == 'uv' %}` AND `{% if package_manager == 'pixi' %}` blocks
6. **CUDA versions**: Ensure CUDA version in UV index URL matches Pixi conda CUDA version
7. **PYTHONNOUSERSITE**: NEVER remove this from Pixi config—it's critical for isolation
8. **Breaking changes**: Template updates affect all future generated projects

---
> Source: [nishide-dev/ml-research-template](https://github.com/nishide-dev/ml-research-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
