## rinbp

> name: "Jupyter Notebook Dependency Management"

name: "Jupyter Notebook Dependency Management"
description: "Best practices for managing dependencies in Jupyter notebooks"
tags: ["jupyter", "python", "notebook", "dependencies", "packages"]
fileMatch: ["**/*.ipynb"]
severity: "suggestion"

rule: |
  # Dependency Management in Jupyter Notebooks

  ## Import Statements
  - Place all imports at the beginning of the notebook
  - Group imports by standard library, third-party, and local modules
  - Use explicit imports (`from module import specific_function`) for clarity
  - Avoid wildcard imports (`from module import *`)
  - Add comments explaining the purpose of less common libraries

  ## Version Pinning
  - Include a cell that checks/prints package versions for critical dependencies
  - Consider using `%pip install package==version` for critical dependencies
  - Document minimum required versions for key packages
  - Include a `requirements.txt` or `environment.yml` file with the notebook
  - Use virtual environments for notebook projects

  ## Dependency Installation
  - Include installation instructions for non-standard packages
  - Use `!pip install` or `%pip install` for missing dependencies
  - Consider adding a setup cell that installs required packages
  - Check for package availability before using (`try/except ImportError`)
  - Document any system dependencies (e.g., CUDA for GPU acceleration)

  ## Reproducibility
  - Use package version pinning for reproducible analysis
  - Document the Python version used for development
  - Consider using containerization (Docker) for complex environments
  - Include a conda/pip environment export if applicable
  - Test notebook in a clean environment to verify dependency list is complete

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nibzard)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nibzard)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
