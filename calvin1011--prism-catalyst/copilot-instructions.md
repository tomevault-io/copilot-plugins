## prism-catalyst

> Python style and ML service conventions for data pipeline and ML code


# Python and ML Service Conventions

## Language and Layout
- Python 3.10+. Use type hints for function signatures and public APIs.
- ML code lives under **ml-models/** (sentiment, prediction, optimization). ETL and ingestion under **backend/data-pipeline/**.
- Prefer FastAPI for ML/API services. Use a single `requirements.txt` or pyproject.toml per service with pinned versions.

## Data and ML Stack
- Pandas/NumPy for manipulation; scikit-learn for classical ML; PyTorch or TensorFlow for deep learning per README.
- Keep model loading and inference separate from training scripts. Target inference latency &lt; 200ms per prediction where possible.
- Use environment variables or config files for paths and keys; never commit API keys or model weights to the repo.

## Code Quality
- Use a consistent formatter (e.g. Black) and linter (e.g. Ruff or flake8). Docstrings for public functions and classes.
- Prefer small, testable functions. Use pytest for tests under `tests/` or service-local `tests/` directories.

```python
# BAD: bare except, no types
def get_sentiment(text):
    return model(text)

# GOOD
def get_sentiment(text: str) -> float:
    """Return sentiment score in [-1, 1] for financial text."""
    if not text or not text.strip():
        raise ValueError("text must be non-empty")
    return model(text)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/calvin1011)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/calvin1011)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
