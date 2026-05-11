## storyboardui2

> This document provides guidelines for AI agents working on the Storyboard Maker application.

# AGENTS.md - Storyboard Maker Development Guide

This document provides guidelines for AI agents working on the Storyboard Maker application.

## Project Overview

A Python/PyQt6 desktop application that interfaces with ComfyUI for AI image generation. Enables filmmakers to create storyboards with camera angle control, LoRA integration, and reference image support.

**Tech Stack**: Python 3.10+, PyQt6 6.6+, requests 2.31+, Pillow 10.0+, jsonschema 4.19+, ReportLab 4.0+

## Build Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run application (from project root)
python -m storyboard_app.main

# Development mode with hot reload (if implemented)
python -m storyboard_app.main --dev

# Linting
flake8 storyboard_app/ --max-line-length=100
black storyboard_app/ --line-length 100
isort storyboard_app/ --profile black

# Type checking
mypy storyboard_app/

# Tests
pytest tests/                    # Run all tests
pytest tests/ -v               # Verbose output
pytest tests/ -k "test_name"   # Run single test by name
pytest tests/ --tb=short       # Short traceback
pytest tests/ --cov=storyboard_app  # With coverage
```

## Code Style Guidelines

### Imports

```python
# Standard library first, then third-party, then local
import json
import pathlib
from pathlib import Path
from typing import Any, Optional

import jsonschema
import requests
from PyQt6.QtCore import QObject, pyqtSignal
from PyQt6.QtWidgets import QMainWindow, QWidget

from .config import Config
from .models.template import Template
```

**Rules**:
- Use absolute imports for package consistency
- Sort imports within each group alphabetically
- Avoid wildcard imports (`from x import *`)
- Use `from typing import ...` for type hints, not typing module prefix

### Formatting

- **Line length**: 100 characters maximum
- **Indentation**: 4 spaces (no tabs)
- **Blank lines**: 2 between class definitions, 1 between function definitions
- **Quotes**: Double quotes for strings, single quotes only when containing double quotes
- **Trailing commas**: Required for multi-line lists/dicts

### Type Hints

```python
# Always use type hints for function signatures
def load_template(self, path: Path) -> Optional[Template]:
    ...

# Use Literal for enum-like strings
from typing import Literal
EngineType = Literal["qwen", "z_image", "flux", "sd"]

# Use TypedDict for config structures
from typing import TypedDict
class ComfyUIConfig(TypedDict):
    server_url: str
    timeout: int
    max_retries: int

# Avoid Any - be specific about types
# BAD: def process(data: Any) -> Any:
# GOOD: def process(data: dict[str, Any]) -> list[str]:
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Modules | `snake_case` | `template_loader.py` |
| Classes | `PascalCase` | `StoryboardPanel` |
| Functions | `snake_case` | `load_workflow()` |
| Variables | `snake_case` | `comfyui_client` |
| Constants | `SCREAMING_SNAKE_CASE` | `DEFAULT_TIMEOUT` |
| Private methods | `_snake_case` | `_validate_schema()` |
| Qt widgets | Suffix with widget type | `MainWindow`, `AngleSelector` |

### Error Handling

```python
# Use custom exceptions hierarchy
class StoryboardError(Exception):
    """Base exception for application errors."""
    pass

class TemplateError(StoryboardError):
    """Template loading or validation errors."""
    pass

class ComfyUIError(StoryboardError):
    """ComfyUI API communication errors."""
    pass

# Handle errors with specific exceptions, not bare except
try:
    template = self._loader.load(path)
except TemplateError as e:
    self._logger.error(f"Failed to load template: {e}")
    return None
except OSError as e:
    raise ComfyUIError(f"File system error: {e}") from e

# Always use context managers for resources
with self._client.session() as session:
    response = session.post(url, json=payload)
```

### PyQt6 Patterns

```python
# Use modern signal syntax (PyQt6)
from PyQt6.QtCore import QObject, pyqtSignal, pyqtSlot

class ImageGenerator(QObject):
    progress = pyqtSignal(int)  # Public signal
    finished = pyqtSignal(list[Path])
    
    @pyqtSlot(str)
    def generate(self, prompt: str) -> None:
        ...

# Widget creation - use classes, not lambdas
# BAD: button.clicked.connect(lambda: self.on_click())
# GOOD: button.clicked.connect(self.on_click)

# Use QApplication.instance() for global access
app = QApplication.instance()
```

### File Organization

```
storyboard_app/
├── main.py              # Entry point
├── app.py               # Application class
├── config.py            # Configuration management
├── core/
│   ├── __init__.py
│   ├── template_loader.py
│   ├── workflow_builder.py
│   ├── comfyui_client.py
│   ├── angle_library.py
│   ├── export_manager.py
│   ├── prompt_builder.py
│   └── session_manager.py
├── models/
│   ├── __init__.py
│   ├── template.py
│   ├── parameter.py
│   ├── lora.py
│   └── image_input.py
├── ui/
│   ├── __init__.py
│   ├── main_window.py
│   ├── panels/
│   │   ├── __init__.py
│   │   ├── template_selector.py
│   │   ├── settings_panel.py
│   │   └── ...
│   └── widgets/
│       ├── __init__.py
│       ├── panel_slot.py
│       └── value_widgets.py
├── templates/           # Built-in templates
└── data/                # Schema, angle definitions
```

### Documentation

```python
class TemplateLoader:
    """Discovers, loads, and validates workflow templates.
    
    Supports both built-in templates from the application bundle
    and user-defined templates from the user_templates directory.
    """
    
    def load(self, path: Path) -> Template:
        """Load a template from the specified path.
        
        Args:
            path: Path to the template JSON file.
            
        Returns:
            Validated Template object.
            
        Raises:
            TemplateError: If the file cannot be read or is invalid.
        """
        ...
```

## Testing Guidelines

- Place tests in `tests/` directory, mirror source structure
- Use `pytest` with `pytest-qt` for widget testing
- Mock ComfyUI API responses with `requests-mock` or similar
- Test edge cases: empty templates, invalid configs, network failures
- Use fixtures for common setup (app instance, mock client)

## Important Paths

- Project root: `/Users/npittas/StoryboardUI` (macOS) or `Z:\Python\StoryboardUI` (Windows)
- Templates: `storyboard_app/templates/`
- User templates: `storyboard_app/user_templates/`
- Output: `storyboard_app/output/`
- Sessions: `sessions/` (relative to output directory parent)
- Config file: `storyboard_app/config.json`

## Key Features

### Session Management
- `SessionManager` class in `core/session_manager.py`
- Sessions store panel images, notes, and metadata
- Structure: `sessions/<name>/session.json` + `sessions/<name>/images/`

### PDF Export
- Grid layouts: 2x3, 2x2, 3x3, 1x1, 3x2 presets
- Panel numbers and optional notes
- Uses ReportLab for PDF generation

### Panel Features
- Notes text field under each panel
- Metadata storage for generation parameters
- Context menu with "View Metadata..." option

### Recent Changes (Jan 2026) — Agent Guidance

These notes help agents interacting with the codebase understand new signals, metadata fields, and UI behaviors introduced in recent updates.

- Signals and Context Menu:
    - `ZoomableCanvasView` now emits `import_image_requested(int)` and `reuse_parameters_requested(int, dict)` in addition to existing signals. Agents should listen/forward these when integrating features.
    - `StoryboardGrid` forwards these signals as `import_image_requested` and `reuse_parameters_requested` too.

- Panel API changes:
    - `StoryboardGrid.set_panel_image(index: int, image_path: Path, metadata: Optional[dict] = None)` now accepts an optional `metadata` dict which will be stored on the panel via `PanelGraphicsItem.set_metadata()` before the image is set.
    - `PanelGraphicsItem` stores an `_image_history` of `(Path, Optional[dict])` tuples and exposes `get_image_history()` and `get_metadata()` helpers.

- Metadata fields captured for newer generations (saved with each panel):
    - `template`: Template name used
    - `parameters`: Parameter values (seed, steps, cfg, etc.)
    - `loras`: LoRA slot settings
    - `prompts`: Collected prompt texts (positive/negative)
    - `angle`: Camera angle token (string)
    - `angle_enabled`: Whether the angle checkbox was enabled (bool)
    - `next_scene_enabled`: Whether the "Next Scene" prefix was enabled (bool)
    - `seed`: Numerical seed
    - `image_paths`: Mapping of image input names to absolute paths (strings)

- Angle selector enhancement:
    - `AngleSelector` has a new method `set_angle_by_token(token: str) -> bool` which selects an angle by token and enables the angle checkbox. Agents that programmatically set angles should call this helper when restoring state.

- Re-use Parameters behavior:
    - Right-clicking a panel and choosing "Re-use Parameters" triggers `reuse_parameters_requested` with the panel index and stored metadata. The main window handles switching tabs, selecting templates, applying parameters, restoring reference image inputs, and setting angle/next-scene flags.

Agents should prefer the provided helper methods and signals rather than directly manipulating widgets or internal attributes; this keeps behavior consistent and avoids breaking UI state.

---
> Source: [NickPittas/StoryboardUI2](https://github.com/NickPittas/StoryboardUI2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
