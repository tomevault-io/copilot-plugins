## tesis-habla-imaginada

> Research code for **imagined speech classification from EEG signals** (CIENTIBECA thesis). Implements deep learning models (EEGNet, MLP, ShallowConvNet, DeepConvNet) and classical ML (ESMB_BR ensemble).

# AGENTS.md - CIENTIBECA EEG Speech Classification

## Project Overview

Research code for **imagined speech classification from EEG signals** (CIENTIBECA thesis). Implements deep learning models (EEGNet, MLP, ShallowConvNet, DeepConvNet) and classical ML (ESMB_BR ensemble).

**Key Technologies**: Python 3.11, PyTorch, scikit-learn, scipy, numpy, pywt

---

## Build / Run Commands

### Environment Setup
```bash
conda env create -f environment.yml
conda activate eeg-speech
```

### Running Experiments
```bash
cd src
python baseline_EEGNet.py              # EEGNet baseline
python experimento_embd_bf.py          # ESMB_BR classical ML
python preprocessing.py                 # Preprocessing pipeline
python augmentation_v2.py              # Data augmentation
python run_s01_kfold.py                # Single subject k-fold

# Debug in VSCode: "Python Debugger: Current File" configuration
```

### No Testing Framework
This codebase has **no test suite** (pytest, unittest, etc.). For development:
- Test interactively in notebooks
- Run individual scripts to verify behavior
- Use `python -c "import module; module.function()"` for quick checks

---

## Code Style Guidelines

### Import Organization
```python
# 1. Standard library
import os, json, time
from pathlib import Path
from typing import List, Optional, Dict

# 2. Third-party
import numpy as np
import torch
import torch.nn as nn
from sklearn.metrics import classification_report
from scipy.signal import butter, filtfilt

# 3. Local imports (relative in src/)
# from .models import EEGNet
```

### Naming Conventions
This codebase uses **Spanish for variables/logic** and **English for technical terms**:

```python
# Spanish (business logic)
etiquetas = labels
perdida_total = total_loss
cargador_entrenamiento = train_loader
semilla = seed

# English (technical terms)
EEGNet, MLP, CrossEntropyLoss
trainer.ajustar(), evaluator.probar()
X_train, Y_train
```

### Class/Method Naming
```python
class Entrenador: ...        # Spanish class names
def ajustar(self, ...):       # Spanish verbs for methods
def probar(self, dataloader):
def guardar(self, path):
```

### Section Separators
```python
####################################################################################################################################
# BLOCK NAME
####################################################################################################################################
```

### Class Docstrings
```python
class EEGNet(nn.Module):
    """
    EEGNet (PyTorch). Diseño inspirado en Lawhern et al.
    Input: (batch, C, T) where C = n_channels, T = n_timepoints.
    Key args:
      - in_ch: number of EEG channels (C)
      - n_classes: number of output classes
    """
```

---

## Error Handling Patterns
```python
try:
    torch.save(self.modelo.state_dict(), nombre_modelo_salida)
except Exception as e:
    print(f"[Entrenador] Warning: no se pudo guardar: {e}")

# Graceful degradation
if hasattr(self.modelo, "apply_max_norm"):
    try:
        self.modelo.apply_max_norm()
    except Exception as e:
        print(f"[Entrenador] Warning apply_max_norm fallo: {e}")
```

---

## Data Format Conventions

### EEG Array Shapes
```python
# Raw trial: (n_channels, n_timepoints)
# Batched: (n_trials, n_channels, n_timepoints)
# After feature extraction: (n_trials, n_features)

# Labels: Y shape (n_trials, 3) = [modalidad, estímulo, artefacto]
# Extended Y: (n_trials, 6) = [modalidad, stim, artefact, window, band, fts]
```

### Label Values
```python
# Modalidad: 0 = imaginada, 1 = pronunciada
# Estímulo: 1-5 (A,E,I,O,U), 6-11 (up,down,left,right,forward,back)
# Artefacto: 0 = limpio, 1 = parpadeo
```

---

## GPU/CUDA Handling
```python
if device is None:
    self.device = 'cuda' if torch.cuda.is_available() else 'cpu'
else:
    if device == 'cuda' and not torch.cuda.is_available():
        print("Advertencia: CUDA no disponible, usando CPU.")
        self.device = 'cpu'
self.device = torch.device(self.device)

# Reproducibility
torch.backends.cudnn.deterministic = True
torch.backends.cudnn.benchmark = False
```

---

## Reproducibility
```python
np.random.seed(seed)
torch.manual_seed(seed)
torch.cuda.manual_seed_all(seed)
random.seed(seed)
os.environ['PYTHONHASHSEED'] = str(seed)
```

---

## Git Ignore (DO NOT COMMIT)
- Raw data files (`.mat`, `.npz` in `data/`)
- Processed data files (`data/processed/`)
- Model checkpoints (`.pt`, `.pth`)
- Experiment results (`experiments/`, `results/`)
- `.env` files, `features/` directory

---

## Common Tasks

### Adding a new model
1. Add class to `src/models.py`
2. Follow EEGNet/MLP pattern with `__init__`, `forward`, optional `apply_max_norm()`
3. Add experiment runner in `src/`

### Debugging training
1. Set `MAX_SUBJECTS = 1` in experiment runner
2. Set `N_SEEDS = 1`, `K_FOLDS = 3` for quick iteration
3. Use `SAVE_BEST_MODEL = False` to skip disk writes

---
> Source: [enzocatorano/tesis_habla_imaginada](https://github.com/enzocatorano/tesis_habla_imaginada) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
