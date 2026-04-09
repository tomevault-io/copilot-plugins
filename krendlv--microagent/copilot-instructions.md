## microagent

> Open-source Python CLI + MCP server for automated microscopy image segmentation, evaluation, training, and reporting. Works standalone or with any MCP-compatible LLM.

# MicroAgent — Agentic Microscopy Image Analysis

## What This Is
Open-source Python CLI + MCP server for automated microscopy image segmentation, evaluation, training, and reporting. Works standalone or with any MCP-compatible LLM.

## Build & Test
```
uv sync                           # Install deps
uv sync --extra dev               # Install dev deps
uv run pytest                     # Run tests
uv run pytest --cov=microagent    # With coverage
uv run ruff check src/            # Lint
uv run ruff format src/           # Format
uv run microagent --help          # CLI help
```

## Architecture
src/microagent/
  cli.py          → Typer CLI entry point, delegates to core/
  mcp_server.py   → FastMCP server wrapping same functions as CLI
  core/
    inspect.py    → Image loading, QC, statistics
    segment.py    → Inference: CellPose, StarDist, μSAM (abstract Segmenter base)
    evaluate.py   → Metrics via stardist.matching + scipy fallback
    train.py      → Fine-tuning (CellPose cpsam primary)
    optimize.py   → Model selection heuristics + Optuna HPO
  viz/
    overlays.py   → Segmentation overlay composites
    plots.py      → Metric charts (matplotlib Agg backend only)
    report.py     → HTML report generator (Jinja2 + base64)
  fair/
    provenance.py → Auto-collected run metadata
    tracking.py   → experiments.jsonl logger
  project/
    knowledge.py  → project.yaml creation and parsing

## Code Conventions
- Python >=3.10, type hints on all public functions
- pathlib.Path over os.path everywhere
- rich for terminal output, never bare print()
- NumPy-style docstrings
- No GUI dependencies in core (matplotlib Agg only)
- Optional deps guarded with try/except ImportError
- All file paths via CLI args or project.yaml, never hardcoded
- Every function that does I/O returns a typed dataclass or dict
- Tests use synthetic images, never real data in the repo

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/krendlV)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/krendlV)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
