## face-age-estimator

> **Last Updated:** Feb 6, 2026

# AGENTS.md - Face Age Estimator Development Guide

**Version:** 1.0  
**Last Updated:** Feb 6, 2026  
**Project:** Real-Time Age Detection from Facial Images Using Transfer Learning

This document provides comprehensive guidelines for agentic coding agents working on the Face Age Estimator project.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Build, Lint, and Test Commands](#build-lint-and-test-commands)
3. [Code Style Guidelines](#code-style-guidelines)
4. [Project Structure](#project-structure)
5. [Development Workflow](#development-workflow)
6. [Testing Standards](#testing-standards)
7. [ML/AI Specific Guidelines](#mlai-specific-guidelines)

---

## Project Overview

This is a **machine learning project** for age and gender estimation from facial images using transfer learning with ResNet50 and MTCNN face detection. The system uses:
- **Dataset:** UTKFace (age 0-116, with gender and ethnicity labels)
- **Backend:** FastAPI for model serving
- **Frontend:** React (planned)
- **ML Framework:** TensorFlow/Keras
- **Face Detection:** MTCNN
- **Python Version:** 3.12+

**Key Technical Concepts:**
- Hierarchical gender-specific models
- Label Distribution Age Encoding (LDAE)
- Transfer learning with ResNet50 backbone
- Multi-task CNN for face detection and alignment

---

## Build, Lint, and Test Commands

### Environment Setup
```bash
# Install dependencies (using uv package manager)
uv sync

# Activate virtual environment
# Windows:
.venv\Scripts\activate
# Linux/Mac:
source .venv/bin/activate
```

### Running the Application
```bash
# Start FastAPI backend server
uvicorn src.api.main:app --reload

# Start with specific host/port
uvicorn src.api.main:app --host 0.0.0.0 --port 8000 --reload
```

### Linting and Formatting
```bash
# Format code with Black
black src/ tests/

# Format specific file
black src/core/training.py

# Check formatting (dry-run)
black --check src/

# Lint with Ruff
ruff check src/ tests/

# Lint and auto-fix
ruff check --fix src/

# Lint specific file
ruff check src/core/inference.py
```

### Testing
```bash
# Run all tests
pytest

# Run tests with verbose output
pytest -v

# Run tests with coverage
pytest --cov=src --cov-report=html

# Run a specific test file
pytest tests/test_preprocessing.py

# Run a single test function
pytest tests/test_preprocessing.py::test_face_alignment

# Run tests matching a pattern
pytest -k "test_age"

# Run tests and show print statements
pytest -s
```

### Jupyter Notebooks
```bash
# Start Jupyter Lab (for exploration and analysis)
jupyter lab

# Convert notebook to Python script
jupyter nbconvert --to script notebooks/exploration.ipynb
```

---

## Code Style Guidelines

### General Principles
- **PEP 8 Compliant:** Follow Python PEP 8 style guide
- **Type Hints Required:** Use type annotations for all function signatures
- **Docstrings Required:** Use Google-style docstrings for all public functions and classes
- **Line Length:** Maximum 88 characters (Black default)
- **Imports:** Organized and sorted (isort compatible)

### Import Order
```python
# 1. Standard library imports
import os
import sys
from pathlib import Path
from typing import List, Dict, Optional, Tuple

# 2. Third-party imports
import numpy as np
import pandas as pd
import tensorflow as tf
from fastapi import FastAPI, UploadFile
from mtcnn import MTCNN

# 3. Local application imports
from src.core.preprocessing import preprocess_image
from src.core.face_detection import detect_face
```

**IMPORTANT: Never use `importlib.util` for importing local modules.** Always use standard Python imports (`from module import function`). Dynamic imports with `importlib` add unnecessary complexity and provide no benefit for local project files.

### Naming Conventions
- **Variables/Functions:** `snake_case` (e.g., `preprocess_image`, `age_prediction`)
- **Classes:** `PascalCase` (e.g., `AgeEstimator`, `FaceDetector`)
- **Constants:** `UPPER_SNAKE_CASE` (e.g., `MAX_AGE`, `IMAGE_SIZE`)
- **Private members:** Prefix with single underscore (e.g., `_internal_helper`)
- **Module-level "private" items:** Prefix with single underscore (e.g., `_validate_input`)

### Type Annotations
```python
# Always use type hints
def predict_age(image: np.ndarray, model: tf.keras.Model) -> Tuple[int, float]:
    """Predict age from preprocessed face image.
    
    Args:
        image: Preprocessed face image array of shape (224, 224, 3).
        model: Trained Keras model for age prediction.
        
    Returns:
        Tuple of (predicted_age, confidence_score).
    """
    prediction = model.predict(np.expand_dims(image, axis=0))
    age = int(np.argmax(prediction))
    confidence = float(np.max(prediction))
    return age, confidence
```

### Docstring Format (Google Style)
```python
def preprocess_face(
    image_path: str,
    target_size: Tuple[int, int] = (224, 224),
    normalize: bool = True
) -> Optional[np.ndarray]:
    """Preprocess face image for model inference.
    
    Performs face detection, alignment, cropping, resizing, and
    optional pixel normalization.
    
    Args:
        image_path: Path to the input image file.
        target_size: Target dimensions for resized image (width, height).
        normalize: Whether to normalize pixel values to [0, 1].
        
    Returns:
        Preprocessed image array of shape (height, width, 3), or None
        if no face is detected.
        
    Raises:
        FileNotFoundError: If image_path does not exist.
        ValueError: If target_size dimensions are invalid.
    """
    pass
```

### Error Handling
```python
# Use specific exception types
try:
    image = load_image(path)
except FileNotFoundError as e:
    logger.error(f"Image not found: {path}")
    raise
except ValueError as e:
    logger.error(f"Invalid image format: {e}")
    return None

# Provide context in error messages
if age < 0 or age > 116:
    raise ValueError(
        f"Age {age} is out of valid range [0, 116] for UTKFace dataset"
    )
```

### Logging
```python
import logging

# Use module-level logger
logger = logging.getLogger(__name__)

# Log levels:
# - DEBUG: Detailed diagnostic information
# - INFO: General informational messages
# - WARNING: Warning messages for recoverable issues
# - ERROR: Error messages for serious problems
# - CRITICAL: Critical errors that may cause failure

logger.info(f"Loading model from {model_path}")
logger.warning(f"No face detected in image {image_id}")
logger.error(f"Model inference failed: {error}")
```

### Constants and Configuration
```python
# Define constants at module level
IMAGE_SIZE = (224, 224)
MAX_AGE = 116
MIN_AGE = 0
CONFIDENCE_THRESHOLD = 0.7

# Use dataclasses or Pydantic for configuration
from dataclasses import dataclass

@dataclass
class ModelConfig:
    """Configuration for age estimation model."""
    image_size: Tuple[int, int] = (224, 224)
    batch_size: int = 32
    learning_rate: float = 0.001
    epochs: int = 50
```

### File Paths in Scripts
**IMPORTANT: Always use `__file__` relative paths in standalone scripts** (files in `scripts/` directory that can be run directly). Never use relative string paths like `"data/utk_face"` as they depend on the current working directory.

```python
# Good: Path relative to script location
from pathlib import Path

SCRIPT_DIR = Path(__file__).parent  # scripts/
PROJECT_ROOT = SCRIPT_DIR.parent     # face_age_estimator/
DATASET_PATH = PROJECT_ROOT / "data" / "utk_face"

# Bad: Depends on working directory
DATASET_PATH = "data/utk_face"  # Only works if run from project root!
```

This ensures scripts work correctly regardless of where they are executed from.

---

## Project Structure

```
face_age_estimator/
├── src/
│   ├── api/              # FastAPI application
│   │   ├── main.py       # FastAPI app initialization
│   │   └── routes.py     # API endpoints
│   ├── core/             # Core ML logic
│   │   ├── face_detection.py    # MTCNN face detection
│   │   ├── preprocessing.py     # Image preprocessing
│   │   ├── training.py          # Model training
│   │   ├── inference.py         # Model inference
│   │   └── exploration.py       # Data exploration
│   └── utils/            # Utility functions
├── tests/                # Unit and integration tests
├── models/               # Saved model files (.h5, .keras)
├── data/                 # Dataset storage (not committed)
├── notebooks/            # Jupyter notebooks for experiments
├── pyproject.toml        # Project dependencies
└── README.md             # Project documentation
```

---

## Testing Standards

### Test File Organization
- Mirror `src/` structure in `tests/`
- Name test files as `test_<module_name>.py`
- Use descriptive test function names: `test_<function>_<scenario>`

### Test Structure
```python
import pytest
import numpy as np
from src.core.preprocessing import preprocess_image

def test_preprocess_image_valid_input():
    """Test preprocessing with valid face image."""
    image = np.random.randint(0, 255, (500, 500, 3), dtype=np.uint8)
    result = preprocess_image(image)
    
    assert result is not None
    assert result.shape == (224, 224, 3)
    assert result.dtype == np.float32
    assert 0.0 <= result.min() <= result.max() <= 1.0

def test_preprocess_image_no_face_detected():
    """Test preprocessing when no face is detected."""
    blank_image = np.zeros((500, 500, 3), dtype=np.uint8)
    result = preprocess_image(blank_image)
    
    assert result is None

@pytest.fixture
def sample_model():
    """Fixture providing a mock model for testing."""
    return create_test_model()
```

### Test Coverage
- Aim for >80% code coverage
- Focus on core logic and edge cases
- Mock external dependencies (file I/O, API calls)

---

## ML/AI Specific Guidelines

### Model Development
- Use **TensorFlow/Keras** for all models
- Save models in `.h5` or `.keras` format
- Version models with descriptive names: `age_model_resnet50_v1.h5`
- Log training metrics using TensorBoard or Weights & Biases

### Data Handling
- **Never commit large datasets** to Git (use .gitignore)
- Store data in `data/` directory
- Use relative paths: `Path("data/UTKFace")`
- Validate data integrity before training

### Preprocessing Standards
```python
# Consistent image preprocessing pipeline:
# 1. Face detection with MTCNN
# 2. Face alignment using landmarks
# 3. Crop to face region
# 4. Resize to (224, 224)
# 5. Normalize pixel values to [0, 1]
# 6. Convert to float32
```

### Age Encoding (LDAE)
- Use **Label Distribution Age Encoding** for training
- Increase standard deviation linearly with age
- Implement as Gaussian distribution around true age

### Model Architecture
- **Backbone:** ResNet50 (pretrained on ImageNet)
- **Hierarchical approach:** Gender classifier → Gender-specific age models
- **Output layers:** Separate for age (regression) and gender (binary classification)

### Inference Pipeline
```python
# Standard inference flow:
# 1. Load image
# 2. Detect face with MTCNN
# 3. Preprocess face
# 4. Predict gender
# 5. Use gender-specific age model
# 6. Return age + confidence
```

---

## Additional Notes

### Security and Privacy
- **Do not store face images** by default (per NFR-3)
- Log metadata only (timestamp, prediction, confidence)
- Use HTTPS/TLS for API communication
- Require authentication for admin functions

### Performance Considerations
- Batch process images when possible
- Cache model in memory (don't reload on each request)
- Use GPU acceleration if available
- Optimize image loading and preprocessing

### Documentation
- Keep this AGENTS.md updated with new conventions
- Document architectural decisions in code comments
- Maintain API documentation with FastAPI auto-docs

---

**For questions or clarifications, refer to:**
- README: `README.md`
- Dependencies: `pyproject.toml`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dltvn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dltvn)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
