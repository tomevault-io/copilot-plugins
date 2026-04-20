## daggr

> - AVOID inline comments

# CLAUDE.md

## Code Style

- AVOID inline comments

## Commands

```bash
pip install -e .[dev]       # Install dev dependencies
pytest                      # Run tests
ruff check --fix --select I && ruff format  # Lint and format
```

## Architecture

DAG-based workflow library connecting Gradio Spaces, ML models, and Python functions.

### Core Files

- **`graph.py`**: `Graph` class - holds nodes, edges, launches server
- **`node.py`**: `GradioNode`, `FnNode`, `InferenceNode`, `InteractionNode`
- **`port.py`**: `Port`, `ScatteredPort`, `GatheredPort` for parallel execution
- **`edge.py`**: `Edge` class for node connections
- **`executor.py`**: Executes workflow nodes
- **`server.py`**: FastAPI server with WebSocket for real-time UI updates
- **`state.py`**: Session state persistence with SQLite
- **`frontend/`**: Svelte frontend for workflow visualization

### Node Definition Pattern

Nodes use `inputs` and `outputs` dicts:

```python
node = GradioNode(
    space_or_url="owner/space",
    api_name="/endpoint",
    inputs={
        "param": gr.Textbox(label="Label"),  # UI input
        "other": other_node.output_port,      # Port connection
        "fixed": "constant_value",            # Fixed value
    },
    outputs={
        "result": gr.Audio(label="Result"),
        "hidden": gr.Text(visible=False),     # Hidden output
    },
)
```

### Key Patterns

1. **Port access**: `node.port_name` returns a `Port` for connections
2. **Graph creation**: `Graph(nodes=[...])` auto-wires from port connections in inputs
3. **Internal attrs**: Use `_` prefix (`_name`, `_fn`) to avoid port name conflicts

## Issue Workflow

1. Create branch: `git checkout -b fix-issue-NUMBER`
2. Implement fix
3. Run: `pytest && ruff check --fix --select I && ruff format`
4. Do not commit - leave for user

---
> Source: [gradio-app/daggr](https://github.com/gradio-app/daggr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
