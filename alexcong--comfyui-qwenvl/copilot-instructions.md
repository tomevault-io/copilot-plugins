## comfyui-qwenvl

> Core logic lives in `nodes.py`, which defines the Qwen multimodal and text nodes plus helper utilities for CUDA memory and Hugging Face downloads. Packaging metadata is in `pyproject.toml`, dependencies in `requirements.txt`, and reusable launch configs under `config/`. Reference workflows (`workflow/*.json`) demonstrate expected node wiring and ship with screenshots for quick visual checks. Tests mirror the runtime entry points inside `tests/`, while ComfyUI caches downloaded checkpoints in `ComfyUI/models/LLM/`; avoid committing anything below that directory.

# Repository Guidelines

## Project Structure & Module Organization
Core logic lives in `nodes.py`, which defines the Qwen multimodal and text nodes plus helper utilities for CUDA memory and Hugging Face downloads. Packaging metadata is in `pyproject.toml`, dependencies in `requirements.txt`, and reusable launch configs under `config/`. Reference workflows (`workflow/*.json`) demonstrate expected node wiring and ship with screenshots for quick visual checks. Tests mirror the runtime entry points inside `tests/`, while ComfyUI caches downloaded checkpoints in `ComfyUI/models/LLM/`; avoid committing anything below that directory.

## Build, Test, and Development Commands
Install dependencies inside the ComfyUI virtualenv with `pip install -r requirements.txt`. During development, place this repo under `ComfyUI/custom_nodes/ComfyUI_QwenVL` or symlink it there so ComfyUI autoloads the nodes. Use `python -m unittest tests.test_nodes` for the lightweight lifecycle tests, or `pytest tests -k nodes` if you prefer richer failure output. When hacking on scripts directly, export `PYTHONPATH=.` to resolve in-repo imports and set `HF_HOME` if you need a custom cache.

## Coding Style & Naming Conventions
Follow PEP 8 with 4-space indentation, snake_case functions (`_clear_cuda_memory`) and PascalCase node classes (`QwenVL`, `Qwen`). Keep node attributes explicit (`RETURN_TYPES`, `CATEGORY`) so ComfyUI can surface them. Guard optional imports (e.g., `comfy.model_management`) with try/except and release GPU resources via `_maybe_move_to_cpu` before clearing CUDA. Prefer descriptive parameter names over abbreviations and document non-obvious device-handling decisions with short comments.

## Testing Guidelines
`tests/test_nodes.py` relies on the standard `unittest` runner plus dummy processors/models, enabling CPU-only execution. Match that pattern when extending coverage: patch network calls, stub tensors with predictable shapes, and assert both the generated outputs and resource cleanup behaviors. Name files and methods `test_<behavior>` so discovery remains automatic, and skip GPU-specific cases when `torch.cuda.is_available()` is false. Run the suite locally before every PR and paste failures when hardware differences appear.

## Commit & Pull Request Guidelines
History favors concise, imperative subjects (`Add VRAM cleanup helpers`, `Add new Qwen3 models`). Continue that style and add detail in the body only when necessary. Each PR should provide: (1) a summary of the node or workflow change, (2) test evidence (`python -m unittest` output), (3) notes on model downloads or VRAM impact, and (4) updated workflow JSON or screenshots when UI wiring changes. Cross-link GitHub issues and call out any configuration or dependency adjustments that downstream ComfyUI users must perform.

---
> Source: [alexcong/ComfyUI_QwenVL](https://github.com/alexcong/ComfyUI_QwenVL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
