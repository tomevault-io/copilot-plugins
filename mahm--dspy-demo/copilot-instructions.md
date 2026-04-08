## dspy-demo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a DSPy framework demonstration using marimo interactive notebooks. DSPy automates prompt tuning for LLM applications through data-driven optimization rather than manual prompt engineering.

## Development Commands

### Setup
```bash
# Install dependencies
uv sync

# Create .env from template
cp .env.example .env
# Edit .env with your Azure OpenAI or OpenAI credentials
```

### Running marimo notebooks
```bash
# Run a specific notebook
uv run marimo edit notebooks/01_chatbot/01_before_tuning.py

# Or from within the notebook directory
cd notebooks/01_chatbot
uv run marimo edit 01_before_tuning.py
```

### Quality checks
```bash
# Lint check
uv run ruff check .

# Type check (excludes notebooks/)
uv run mypy .
```

## Architecture

### Core Components

**common/config.py**: LM configuration abstraction
- Provides `configure_lm()` for unified Azure OpenAI / OpenAI setup
- Controlled by `PROVIDER_NAME` environment variable ("azure" or "openai")
- Default models: `SMART_MODEL=gpt-4.1`, `FAST_MODEL=gpt-4.1-nano`

**common/model_saver.py**: Model persistence with versioning
- `save_model_with_timestamp()`: Saves DSPy models with timestamp + optional score
- Creates symlink `{prefix}_latest.json` pointing to newest model
- `load_latest_model()`: Loads via the latest symlink
- Models stored in `notebooks/{demo}/artifact/` directories (version controlled)

### DSPy Module Pattern

All DSPy modules follow this structure:

```python
class MySignature(dspy.Signature):
    """Description of task"""
    input_field = dspy.InputField(desc="...")
    output_field = dspy.OutputField(desc="...")

class MyModule(dspy.Module):
    def __init__(self):
        super().__init__()
        self.predictor = dspy.Predict(MySignature)

    def forward(self, input_field: str) -> dspy.Prediction:
        return self.predictor(input_field=input_field)
```

## marimo Notebook Patterns

### Critical: UIElement Display Pattern

marimo requires that the **last expression** in a cell be evaluated for display. These patterns cause nothing to display:

```python
# ❌ WRONG - return statement prevents display
if condition:
    mo.md("content")
else:
    mo.md("")
return

# ❌ WRONG - if/else is a statement, not evaluated
if condition:
    mo.md("content")
else:
    mo.md("")

# ✅ CORRECT - assign to variable and evaluate
if condition:
    display = mo.md("content")
else:
    display = mo.md("")

display  # Last expression gets displayed
```

### Button Trigger Pattern

Buttons must use `mo.state()` triggers, NOT `.value` checking:

```python
# Cell 1: Create trigger and button (depends only on mo - stable)
@app.cell
def _(mo):
    get_trigger, set_trigger = mo.state(0)

    button = mo.ui.button(
        label="Execute",
        on_change=lambda _: set_trigger(lambda v: v + 1)  # Increment on click
    )

    return get_trigger, button

# Cell 2: Process on trigger change
@app.cell
def _(mo, get_trigger, other_dependencies):
    trigger_value = get_trigger()  # Reference trigger to watch for changes

    if trigger_value > 0:
        # Execute action
        result = perform_action()
        display = mo.md(f"Result: {result}")
    else:
        display = mo.md("")

    display  # Last expression displayed
```

**Why this pattern works:**
- Trigger state is created in a cell that depends only on `mo` (stable, won't re-execute)
- If state is in a cell depending on variables that change, `mo.state()` recreates on re-execution, breaking reactivity
- Processing cell watches the trigger getter, auto-executes when trigger changes

### Path Resolution

Notebooks use `__file__` to build absolute paths, ensuring they work regardless of execution directory:

```python
import os
notebook_dir = os.path.dirname(os.path.abspath(__file__))
artifact_dir = os.path.join(notebook_dir, "artifact")
```

## Workflow: 01_chatbot Demo

1. **01_before_tuning.py**: Run baseline chatbot (unoptimized prompts)
2. **02_tuning.py**: Execute MIPROv2 optimization on training data, save optimized model to `artifact/`
3. **03_after_tuning.py**: Load optimized model, compare before/after results

Each notebook is self-contained and can be run from project root or notebook directory.

## Environment Variables

Required in `.env`:

**Azure OpenAI:**
```env
PROVIDER_NAME=azure
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_API_KEY=your-api-key
AZURE_OPENAI_API_VERSION=2025-04-01-preview
```

**OpenAI:**
```env
PROVIDER_NAME=openai
OPENAI_API_KEY=your-api-key
```

Optional overrides:
- `SMART_MODEL`: Default "gpt-4.1"
- `FAST_MODEL`: Default "gpt-4.1-nano"
- `EMBEDDING_MODEL`: Default "text-embedding-3-small"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mahm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mahm)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
