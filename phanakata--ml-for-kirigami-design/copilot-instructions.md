## ml-for-kirigami-design

> Machine learning package for modeling and topology optimization of graphene kirigami using deep learning. Uses CNNs (VGGNet-style) for regression and supervised autoencoders for forward/inverse design.

# Project Context for Claude

## Overview
Machine learning package for modeling and topology optimization of graphene kirigami using deep learning. Uses CNNs (VGGNet-style) for regression and supervised autoencoders for forward/inverse design.

## Project Structure
- `analysis/` - Analysis scripts and notebooks
- `generate_LAMMPS_input/` - Generate atomic configurations for LAMMPS simulations
- `mddata/` - Raw dataset (coarse-grained grid, AIREBO potential)
- `models_regression/` - CNN regression models (TensorFlow)
- `models_supervisedAutoencoder_forwardInverseDesign/` - Supervised autoencoder for design
- `tools/` - Utility functions

## Key Files
- `generate_LAMMPS_input/generate_LAMMPS_configuration_input.ipynb` - Generate LAMMPS input
- `models_supervisedAutoencoder_forwardInverseDesign/supervisedAE_for_kirigamiDesign.ipynb` - Forward/inverse design

## Tech Stack
- Python
- TensorFlow (deep learning)
- scikit-learn (simple ML)
- LAMMPS (molecular dynamics simulations)

## Publications
1. Hanakata et al., Phys. Rev. Lett. 121, 255304 (2018) - CNN for kirigami optimization
2. Hanakata et al., Phys. Rev. Research 2, 042006 (2020) - Supervised autoencoder for design

---
> Source: [phanakata/ML_for_kirigami_design](https://github.com/phanakata/ML_for_kirigami_design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
