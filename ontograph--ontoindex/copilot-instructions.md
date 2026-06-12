## 200-eval

> - **Run tests**: `cd eval && uv run pytest tests/`


# OntoIndex eval harness (Python)

- **Run tests**: `cd eval && uv run pytest tests/`
- **Run with coverage**: `cd eval && uv run coverage run -m pytest tests/ && uv run coverage report`
- **Lint**: `cd eval && uv run ruff check .`
- **Run eval**: `cd eval && uv run python run_eval.py --config configs/<config>.yaml`
- Shared constants live in `eval/constants.py`; tool specs in `eval/tool_registry.py`.
- Error logging uses `utils/errors.py` — set `ONTOINDEX_EVAL_DEBUG=1` for full tracebacks.
- Property-based tests use Hypothesis (`eval/tests/test_property_based.py`).

---
> Source: [ontograph/ontoindex](https://github.com/ontograph/ontoindex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
