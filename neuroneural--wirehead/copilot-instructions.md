## wirehead

> pip install build twine

# PyPI Build Instructions

## Building and publishing to PyPI

### 1. Install build tools
```bash
pip install build twine
```

### 2. Generate distribution packages
```bash
python -m build
```

### 3. Upload to PyPI
```bash
python -m twine upload dist/*
```

### 4. Upload to TestPyPI (for testing before real release)
```bash
python -m twine upload --repository testpypi dist/*
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neuroneural)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/neuroneural)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
