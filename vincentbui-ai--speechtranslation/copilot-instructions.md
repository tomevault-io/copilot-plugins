## speechtranslation

> Guidelines for AI coding agents working on this Vietnamese-English Speech Translation repository.

# AGENTS.md — Speech Translation Project

Guidelines for AI coding agents working on this Vietnamese-English Speech Translation repository.

## Project Overview

This repository contains training and inference code for Speech-to-Text Translation (ST) models using SeamlessM4T for the Vietnamese ↔ English language pair.

## Build / Test / Lint Commands

### Code Quality (via seamless_communication submodule)
```bash
cd src/seamless_communication

# Format code with Black
black src/ tests/

# Type checking with mypy
mypy src/

# Pre-commit hooks
pre-commit run --all-files
```

### Installation
```bash
# Install SeamlessM4T dependencies

## Code Style Guidelines

### Imports
- **Standard library** first, **third-party** second, **local** third
- Group imports with a blank line between groups
- Use absolute imports over relative imports
- Example:
```python
import json
import os
from pathlib import Path
from typing import List, Optional

import torch
import torchaudio
from tqdm import tqdm

from src.metrics import compute_wer
```

### Formatting
- **Black** code formatter (line length: 88 characters)
- **isort** with "black" profile for import sorting
- Use double quotes for strings consistently
- Trailing commas in multi-line structures

### Type Hints
- Use Python 3.8+ typing syntax
- Annotate function parameters and return types
- Use `Optional[Type]` for nullable values
- Use `List[Type]`, `Dict[Key, Value]` from typing module
- Example:
```python
def compute_cer(reference: str, hypothesis: str, normalize: bool = True) -> float:
    ...
```

### Naming Conventions
- **snake_case** for functions, variables, methods
- **PascalCase** for classes
- **SCREAMING_SNAKE_CASE** for constants
- Private methods/functions prefix with underscore: `_helper()`
- Example:
```python
MAX_SAMPLES = 50000
TARGET_SAMPLE_RATE = 16_000

def load_audio(filepath: str) -> tuple[torch.Tensor, int]:
    ...

class MetricsEvaluator:
    def _normalize_text(self, text: str) -> str:
        ...
```

### Docstrings
- Use Google-style docstrings
- Document all public functions and classes
- Include Args, Returns, and Raises sections
- Example:
```python
def evaluate(self, references: List[str], hypotheses: List[str]) -> MetricsResult:
    """
    Evaluate all metrics on the full corpus.

    Args:
        references: List of ground-truth strings.
        hypotheses: List of model output strings (same length).

    Returns:
        MetricsResult with aggregated CER, WER and BLEU.

    Raises:
        ValueError: If references and hypotheses differ in length.
    """
```

### Error Handling
- Use specific exceptions (ValueError, RuntimeError, etc.)
- Raise exceptions with descriptive messages
- Handle expected errors gracefully in inference code
- Use try/except blocks sparingly, only for expected failures
- Example:
```python
if len(ref_chars) == 0:
    if len(hyp_chars) == 0:
        return 0.0
    raise ValueError("Reference is empty but hypothesis is not.")
```

### Comments
- Write docstrings for modules, classes, and public functions
- Use inline comments sparingly, only for non-obvious logic
- Prefix comments with `# ` (space after hash)
- Use section dividers for long files:
```python
# ---------------------------------------------------------------------------
# Helpers
# ---------------------------------------------------------------------------
```

## Project Structure

```
SpeechTranslation/
├── src/
│   ├── metrics.py              # CER/WER/BLEU evaluation metrics
│   ├── llm.py                  # Gemini LLM wrapper
│   ├── early_stopping.py       # Training utilities
│   └── seamless_communication/ # SeamlessM4T model code
├── scripts/
│   ├── prepare_data.py         # JSONL → TSV conversion
│   ├── train_spm.py            # SentencePiece training
│   └── compute_gcmvn.py        # GCMVN statistics
├── inference/
│   ├── seamless_infer.py       # SeamlessM4T inference
│   ├── single_infer.py         # Single audio inference
│   └── batch_infer.py          # Multi-GPU batch inference
├── configs/                    # Training configs
├── datasets/                   # JSONL datasets (metadata)
└── data/                       # TSV manifests
```

## Key Technologies

- **PyTorch / fairseq2** — Deep learning framework
- **torchaudio** — Audio processing
- **sentencepiece** — Text tokenization
- **sacrebleu** — BLEU score computation
- **hydra** — Configuration management
- **unittest** — Testing framework

## Language Considerations

This project handles both **English** and **Vietnamese** text:
- Use Unicode NFC normalization for text comparison
- Vietnamese requires special handling for diacritics
- Use sacrebleu's "char" tokenizer for Vietnamese BLEU scores

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vincentbui-ai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vincentbui-ai)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
