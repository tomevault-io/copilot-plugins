## r2d-pytorch-notebook

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a PyTorch notebook environment configured for repo2docker with CUDA 12 support. It provides a JupyterLab environment with PyTorch and ML libraries, deployable via Binder or Docker.

## Build and Deployment Commands

### Docker Build
```bash
docker build -t pytorch-notebook .
```

### Run Container (Standard)
```bash
docker run -p 8888:8888 pytorch-notebook
```

### Run Container (With GPU Support)
```bash
docker run --gpus all -p 8888:8888 pytorch-notebook
```

### Using repo2docker
```bash
jupyter-repo2docker https://github.com/atrawog/r2d-pytorch-notebook
```

## Architecture

### Docker Configuration
- **Base Image**: `quay.io/jupyter/pytorch-notebook:cuda12-python-3.11.8`
- **User Setup**: Configured for Binder compatibility with UID 1000 (user: jovyan)
- **Python Environment**: 3.11.8 with PyTorch, CUDA 12, matplotlib, seaborn, pandas, scikit-learn, tensorboard
- **Entrypoint**: JupyterLab server on port 8888 with `--ip=0.0.0.0`, `--allow-root`

### Directory Structure
- `/notebooks/` - Jupyter notebook files (e.g., `pytorch_basics.ipynb`)
- All repository contents copied to `$HOME` (jovyan home directory) in container

### Adding Python Packages
Edit the Dockerfile pip install section (lines 39-44) to add new packages. Always use `--no-cache-dir` flag.

### GPU/CUDA Usage
- CUDA 12 support is built into the base image
- GPU detection and usage examples are in `notebooks/pytorch_basics.ipynb`
- Host must have NVIDIA Docker runtime installed for GPU acceleration

## Key Constraints
- Container runs as non-root user (jovyan) for Binder compatibility
- Port 8888 is hardcoded for JupyterLab
- `.dockerignore` excludes model checkpoints, data files, and environment files from image builds

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atrawog)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/atrawog)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
