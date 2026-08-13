## classifry

> This file provides context for Claude Code when working on this project.

# Claude Code Instructions

This file provides context for Claude Code when working on this project.

## Project Overview

Classifry is an email spam classifier using PyTorch neural networks. It's designed as both a learning tool (Jupyter notebook) and a practical command-line/web application.

## Tech Stack

- **Python 3.11+**
- **Package Manager**: uv
- **Deep Learning**: PyTorch
- **ML Utilities**: scikit-learn (TF-IDF, metrics, train/test split)
- **Data Processing**: pandas, numpy, BeautifulSoup
- **Visualization**: matplotlib, seaborn
- **Web Framework**: Flask
- **Testing**: pytest

## Project Structure

```
classifry/
├── pyproject.toml              # Dependencies managed by uv
├── classifier.py               # CLI tool for training and classification
├── app.py                      # Flask web app for email review/labeling
├── spam_classifier_tutorial.ipynb  # Educational Jupyter notebook
├── templates/                  # Jinja2 HTML templates
│   ├── base.html               # Base template with styling
│   ├── index.html              # Upload form
│   └── result.html             # Three-panel email review
├── tests/                      # Unit tests (pytest)
│   ├── test_classifier.py      # Tests for classifier.py
│   ├── test_app.py             # Tests for Flask app
│   └── fixtures/               # Sample email files for testing
├── models/                     # Saved model artifacts
│   ├── spam_classifier.pth     # PyTorch model weights
│   └── tfidf_vectorizer.pkl    # Fitted TF-IDF vectorizer
└── email-data-set/             # Training data
    ├── ham/hard_ham/           # Legitimate emails (251 files)
    └── spam/spam/              # Spam emails (501 files)
```

## Key Commands

```bash
# Install dependencies
uv sync

# Train a new model
uv run python classifier.py train

# Classify text
uv run python classifier.py classify --text "email content"

# Classify a file
uv run python classifier.py classify path/to/email.txt

# Interactive mode
uv run python classifier.py interactive

# Run Jupyter notebook
uv run jupyter notebook spam_classifier_tutorial.ipynb

# Start web app
uv run python app.py

# Run tests
uv run pytest

# Run tests with verbose output
uv run pytest -v
```

## Architecture

The neural network architecture:
- Input: TF-IDF features (5000 dimensions)
- Hidden layers: 256 → 128 → 64 neurons with ReLU activation
- Dropout (0.3) for regularization
- Output: Sigmoid activation for binary classification

## Code Conventions

- Use type hints for function parameters and return values
- Follow PEP 8 style guidelines
- Keep functions focused and well-documented
- Use pathlib.Path for file paths

## Common Tasks

### Adding new preprocessing steps
Edit the `clean_text()` function in `classifier.py`

### Modifying model architecture
Edit the `SpamClassifier` class in `classifier.py`

### Changing training hyperparameters
Use CLI arguments: `--epochs`, `--batch-size`, `--lr`, `--dropout`, `--patience`

### Modifying web app routes

Edit `app.py` - routes are `/`, `/upload`, `/label`

### Modifying email display

Edit templates in `templates/` - `result.html` has three-panel display

### Adding tests

Add tests to `tests/test_classifier.py` or `tests/test_app.py`

## Web App

The Flask web app (`app.py`) provides:

- Email file upload and parsing
- Three-panel display (rendered HTML, underlying content, classifier view)
- Manual labeling to add emails to training dataset
- CSRF protection on all forms

Environment variables:

- `FLASK_SECRET_KEY`: Secret key for sessions (required in production)
- `FLASK_DEBUG`: Set to `true` for debug mode

## Notes

- Dataset is small (752 emails) - expect some overfitting
- Model uses early stopping to prevent overtraining
- TF-IDF vectorizer must be saved alongside model for inference

---
> Source: [nayfusaurus/classifry](https://github.com/nayfusaurus/classifry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
