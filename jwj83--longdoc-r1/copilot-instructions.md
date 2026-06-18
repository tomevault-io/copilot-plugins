## longdoc-r1

> Research-oriented prototype comparing Flat RAG, Direct long-context, and Hierarchical ReAct agent on long-document QA (NarrativeQA subset). Pure Python, no web framework, no test suite.

# AGENTS.md — Long Document Hierarchical Tool-Use Agent

## Project Overview

Research-oriented prototype comparing Flat RAG, Direct long-context, and Hierarchical ReAct agent on long-document QA (NarrativeQA subset). Pure Python, no web framework, no test suite.

## Build / Run Commands

```bash
# Setup
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Run experiment (from workspace root, parent of long_doc_agent/)
python long_doc_agent/experiments/run_experiment.py \
  --train_samples 50 --validation_samples 50 \
  --eval_split validation --top_k 5 --max_steps 8 \
  --decision_model qwen-8b-instruct \
  --read_model qwen-72b-instruct \
  --answer_model qwen-8b-instruct

# Debug / inspect data samples
python long_doc_agent/debug_narrativeqa_samples.py
```

### Environment Variables (required)

```bash
export BASE_URL="https://your-openai-compatible-endpoint/v1"
export API_KEY="your_api_key"
export MODEL_NAME="gpt-4o-mini"
# Optional per-role overrides: DECISION_MODEL_NAME, READ_MODEL_NAME, QA_MODEL_NAME
# Optional per-role endpoints: DECISION_BASE_URL, READ_BASE_URL, QA_BASE_URL
```

### Running a Single Test

There is no formal test framework. To test a component interactively:

```python
python -c "
from llm.client import OpenAICompatibleClient
client = OpenAICompatibleClient()
print(client.generate([{'role': 'user', 'content': 'hello'}]))
"
```

Or add ad-hoc test scripts at the repo root and remove them after use.

## Code Style Guidelines

### Imports

- Always start with `from __future__ import annotations` (line 1 after docstring).
- Group imports: stdlib → third-party → project-local.
- Use **project-relative imports** from the repo root (e.g., `from agent.parser import ...`, NOT `from .parser import ...`).
- `run_experiment.py` adds `PROJECT_ROOT` to `sys.path` so scripts can be run directly.

### Type Hints

- Use type hints on **all function signatures** (parameters and return types).
- Use `typing` module types: `Dict`, `List`, `Any`, `Optional`, `Set`, `Sequence`.
- Use `str | None` style (Python 3.10+) where already in use, but `Optional[str]` is also acceptable.
- Use `Dict[str, Any]` for flexible payload dictionaries.

### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Classes | PascalCase | `HierarchicalReActAgent`, `ToolResult` |
| Functions/Methods | snake_case | `normalize_text`, `execute_tool` |
| Private methods | leading underscore | `_safe_get_text`, `_build_role_client` |
| Variables | snake_case | `final_answer`, `used_tools` |
| Constants | UPPER_SNAKE_CASE | `PROJECT_ROOT` |

### Dataclasses

- Use `@dataclass` for simple data containers (`TreeNode`, `ToolResult`).
- Use `field(default_factory=list)` for mutable defaults.
- Provide `to_dict()` / `from_dict()` methods for JSON serialization.

### Error Handling

- Use `ValueError` for missing configuration (e.g., unset env vars).
- Use `RuntimeError` for operational failures (e.g., LLM request failure).
- Catch broad `Exception` only at experiment boundaries (`_run_single_method_safe`) to prevent one sample from crashing the full run.
- LLM client retries across multiple candidate URLs before raising.
- Tool execution returns `ToolResult(success=False, error=...)` rather than raising.

### Docstrings

- Use triple-quoted docstrings for all modules, classes, and public methods.
- Keep them concise: one-line summary, optional elaboration.
- Module docstrings describe the module's purpose (e.g., `"Base abstractions for environment tools."`).

### Formatting

- No formatter configured (no ruff, black, or yapf). Follow PEP 8 manually.
- Line length: ~100 chars (some longer lines exist, keep reasonable).
- Use 4-space indentation.
- Blank lines: 2 between top-level definitions, 1 between methods.

### Project Structure

```
├── agent/          # ReAct agent (prompt, parser, react_agent)
├── baselines/      # Flat RAG and direct long-context baselines
├── data/           # Dataset loaders (NarrativeQA)
├── env/            # Document environment (read/qa tools, summarization)
├── evaluation/     # Metrics (EM, F1, BLEU, ROUGE-L)
├── experiments/    # Experiment runner (orchestrates all methods)
├── llm/            # OpenAI-compatible HTTP client
├── tools/          # Tool abstractions (BaseTool, ReadTool, QATool)
├── tree/           # Hierarchical document tree (TreeNode, build_tree)
├── outputs/        # Experiment results (auto-created)
└── prompts/        # Prompt templates
```

### Adding New Code

- New tools: subclass `BaseTool` in `tools/`, implement `run(self, env, **kwargs) -> ToolResult`.
- New baselines: follow the pattern in `baselines/`, return `{"prediction": str}`.
- New metrics: add function to `evaluation/metrics.py`, update `evaluate_predictions()`.
- New experiments: add method to `run_experiment.py`'s loop, follow `_run_single_method_safe` pattern.

---
> Source: [jwj83/longdoc_R1](https://github.com/jwj83/longdoc_R1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
