## compmathandaicourse

> This repository contains **reproducible code examples** for the Computational Mathematics and AI course. It will be distributed as a GitHub submodule and used with Google Colab.

# Code Repository - Agent Instructions

## Goal

This repository contains **reproducible code examples** for the Computational Mathematics and AI course. It will be distributed as a GitHub submodule and used with Google Colab.

## Repository Structure

```
/workspace/code/
├── README.md              # Public-facing documentation with Colab links
├── requirements.txt       # Dependencies for all notebooks
├── CLAUDE.md              # This file (agent instructions)
├── {NN}-{topic}.ipynb     # Simple demos (single notebook)
└── {NN}-{topic}/          # Complex demos (subfolder)
    ├── utils.py           # Shared utilities
    ├── models.py          # Network architectures (if needed)
    └── demo.ipynb         # Experimentation notebook
```

## Conventions

### Naming

- **Simple notebooks**: `{NN}-{descriptive-name}.ipynb` (e.g., `01-polynomial-double-descent.ipynb`)
- **Complex subfolders**: `{NN}-{topic}/` (e.g., `07-scientific-ml/`)
- Use lowercase with hyphens, no spaces

### When to Use Simple vs Complex Structure

**Simple** (single self-contained notebook):
- Single demonstration or experiment
- Less than ~5 helper functions
- No shared code between multiple notebooks

**Complex** (subfolder with .py files):
- Multiple related demonstrations
- Substantial shared utilities (data loading, model definitions, training loops)
- Notebooks should focus on **experimentation and key insights**, not boilerplate

### Reproducibility Requirements (Critical)

All code must use **JAX with explicit PRNG keys** for reproducibility:

```python
import jax
import jax.numpy as jnp
from jax import random

# Master PRNG key - ALL randomness derives from this
MASTER_KEY = random.PRNGKey(42)

# Split keys for different random operations
key1, key2 = random.split(MASTER_KEY)
data_key, noise_key = random.split(key1)

# Use explicit keys for all random operations
noise = random.normal(noise_key, shape=(n,))
```

**Never use**:
- `numpy.random.seed()` (global state)
- `np.random.randn()` without explicit seeding

### Notebook Structure

1. **Colab Setup Cell** (first code cell, required)
   ```python
   # Colab setup - run this cell first if using Google Colab
   import sys, os

   if 'google.colab' in sys.modules:
       repo_dir = '/content/CompMathAndAICourse/'
       if not os.path.exists(repo_dir):
           !git clone https://github.com/lruthotto/CompMathAndAICourse.git {repo_dir}
           %pip install -q jax jaxlib

       sys.path.append(repo_dir)
       os.chdir(repo_dir)
       print(f"Running on Colab, repo cloned to {repo_dir}")
   ```

2. **Title & Introduction** (markdown)
   - Clear title matching lecture topic
   - Problem setup and learning objectives

3. **Imports & Setup**
   - All imports at top
   - For complex: `from utils import ...`
   - Plotting style configuration

4. **Configuration**
   - All hyperparameters in one cell
   - Match lecture slides exactly
   - Comment expected results

5. **Data/Problem Setup**
   - Generate or load data
   - Visualize the problem

6. **Main Experiment**
   - Core computation
   - Visualizations matching slides
   - Print key numerical results

7. **Analysis & Summary**
   - Interpretation
   - Key takeaways

### Code Style

- **Efficient**: Use vectorized ops, single-pass algorithms
- **Clear**: Variable names matching mathematical notation in slides
- **Modern**: Use `jnp.trapezoid` not deprecated `jnp.trapz`
- **Documented**: Docstrings for all functions

### Updating the Repository

When adding a new notebook:

1. Create the notebook following conventions above
2. Update `requirements.txt` if new dependencies needed
3. Update `README.md`:
   - Add row to Notebooks table
   - Include Colab badge: `[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/lruthotto/CompMathAndAICourse/blob/main/{NOTEBOOK_PATH})`

### Testing

Before committing:
- Run notebook end-to-end
- Verify results match expected values from slides
- Run twice to confirm reproducibility (same PRNG key → same results)

## Related Commands

- `/publish-code <lecture-number>` - Guided workflow for publishing lecture code

## Source Materials

Lecture code is located in:
- `lectures/lecture{NN}-*/code/`
- `lectures/lecture{NN}-*/codes/`
- `lectures/lecture{NN}-*/demos/`

Slides for context: `lectures/lecture{NN}-*/slides/`

---
> Source: [lruthotto/CompMathAndAICourse](https://github.com/lruthotto/CompMathAndAICourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
