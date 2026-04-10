## pyfade

> pyFADE (Faceted Alignment Datasets Editor) is a desktop GUI application for creating personalized, preference-tailored, multi-faceted datasets for local LLM fine-tuning. It provides an IDE-style PyQt6 workspace backed by SQLite/SQLCipher, with extensible inference providers.

# CLAUDE.md — pyFADE

## Project overview

pyFADE (Faceted Alignment Datasets Editor) is a desktop GUI application for creating personalized, preference-tailored, multi-faceted datasets for local LLM fine-tuning. It provides an IDE-style PyQt6 workspace backed by SQLite/SQLCipher, with extensible inference providers.

**Stack:** Python 3.11, PyQt6, qt-material, SQLAlchemy 2.0, SQLite, pytest/pytest-qt

## Quick commands

```bash
# Run the application
python run.py
python run.py --debug              # with debug logging
python run.py --sample_id <id>     # open specific sample

# Run tests
pytest                             # full suite
pytest --log-cli-level=DEBUG       # with debug output
pytest tests/test_database/        # specific directory

# Code quality (CI requires all to pass)
yapf -i <file>                     # format file in place
pylint --fail-under=9 py_fade      # lint main package
pylint --fail-under=9 tests        # lint tests
```

## Code style rules

- **4 spaces** for indentation, never tabs
- **140-character** line limit (yapf, pylint, ruff all enforce this)
- **yapf** is the source of truth for formatting — run on changed files before committing
- yapf config: Google style, `split_before_named_assigns=false`, `coalesce_brackets=true`
- Type hints throughout the codebase
- f-strings for formatting, except logging calls which use lazy `%` evaluation
- Module-level imports preferred; avoid local imports unless needed for circular dependency avoidance
- All modules, classes, and methods must have docstrings (Google-style, starting on new line after `"""`)
- Never remove existing comments — update them if outdated
- Per-class loggers: `self.log = logging.getLogger(CLASS_NAME)` with appropriate levels
- Never log sensitive information (passwords, API keys)

## Architecture

```
py_fade/
├── app.py              # Main PyFadeApp class, GUI launcher, config wiring
├── app_config.py       # YAML-based config persistence (AppConfig)
├── features_checker.py # Optional dependency detection (must load first)
├── controllers/        # Business logic (export, import, text_generation, dpo, kto, facet_summary)
├── dataset/            # SQLAlchemy ORM models (sample, facet, prompt, completion, dataset)
├── data_formats/       # Import/export formats (DPO, KTO, ShareGPT, LM-Eval)
├── providers/          # LLM backends (ollama, llama_cpp, mock_echo_model, base_provider)
├── gui/                # PyQt6 widgets (widget_*, dialog_*, window_*)
│   ├── components/     # Reusable widgets (token_picker, completion_rating, clickable_facets)
│   └── auxillary/      # UI helpers (google_icon_font, logprobs_to_color, form_fields)
tests/
├── test_database/      # ORM and dataset tests
├── test_controllers/   # Controller logic tests
├── test_logic/         # Data format and business logic tests
├── test_ui/            # PyQt6 widget tests
├── test_integration/   # End-to-end workflow tests
├── helpers/            # Test fixtures and utilities
└── conftest.py         # Shared pytest fixtures
```

## Key patterns

**GUI widgets:** Separate `setup_ui()` for building components, `set_XXX()` methods for populating data. `set_XXX()` branches on a single if-statement: `None`/new object vs existing object.

**Database access:**
```python
session = dataset.get_session()
obj = session.query(ModelClass).filter(...).first()
dataset.commit()
```

**Provider generation:**
```python
response = provider.generate(model_id, prompt: CommonConversation, prefill, temperature=0.7, top_k=40)
# Returns LLMResponse with completion_text and optional logprobs
```

**Import order:** features_checker first (modifies sys.path), then AppConfig, then standard lib, third-party, local.

## Testing conventions

- Default provider is `mock-echo-model` (deterministic, no external deps)
- Tests use `FLAT_PREFIX_SYSTEM`/`FLAT_PREFIX_USER`/`FLAT_PREFIX_ASSISTANT` markers from `py_fade.providers.flat_prefix_template`
- All GUI tests run in Qt offscreen mode (no popups)
- Key fixtures in `conftest.py`: `qt_app` (session), `temp_dataset`, `app_with_dataset`, `ensure_google_icon_font`
- No arbitrary Python scripts — all testing via unit tests and logging only
- Real LLM provider tests must be isolated and never run by default

## CI/CD

GitHub Actions workflow (`.github/workflows/qa.yml`): runs on every push, Python 3.11, Ubuntu. Steps: `pytest` then `pylint --fail-under=9` on both `py_fade` and `tests`.

## UI conventions

- Google Material Design principles with qt-material theming
- Icons via Google Material Symbols font (`py_fade.gui.aux_google_icon_font`, `google_icon_font` singleton)
- Sidebar navigation for facets, tags, export templates; detail views in separate widgets

## Notable features

- Optional SQLCipher encryption for databases
- Token-level logprob tracking and visualization
- Interactive beam search with prefill history
- Complex filter rules (AND/OR) at both DB and Python level
- DPO/KTO/SFT/PPO export formats, ShareGPT import/export
- Encrypted ZIP export via pyzipper

## Changelog

New features should be logged in `Changelog.md` when completed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DanielKluev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DanielKluev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
