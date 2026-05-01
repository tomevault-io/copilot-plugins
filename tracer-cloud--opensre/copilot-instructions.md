## graph-nodes

> LangGraph pipeline architecture and node development


# Graph & Node Development

## Pipeline Architecture

The agent is a LangGraph `StateGraph` over `AgentState` (a `TypedDict` in `app/state.py`).

### Investigation Flow
```
inject_auth → extract_alert → resolve_integrations → plan_actions → investigate → diagnose
                                                          ↑                          │
                                                          └── (loop if recommendations) ──┘
                                                                                     │
                                                                                  publish → END
```

### Chat Flow
```
inject_auth → router → chat_agent ⇄ tool_executor → END
                     → general → END
```

## Key Files

- `app/graph_pipeline.py` — `build_graph()` wires nodes and edges
- `app/routing.py` — conditional edge functions (`route_by_mode`, `route_after_extract`, etc.)
- `app/state.py` — `AgentState` TypedDict + `AgentStateModel` Pydantic validator
- `app/nodes/__init__.py` — barrel exports for all node functions

## Writing a Node

1. Create a subpackage under `app/nodes/` (e.g., `app/nodes/my_step/`)
2. Implement the node function in `node.py`
3. Re-export from the subpackage `__init__.py`
4. Register in `app/nodes/__init__.py` and wire into `graph_pipeline.py`

### Node function pattern:

```python
from langsmith import traceable
from app.output import get_tracker
from app.state import InvestigationState

@traceable(name="node_my_step")
def node_my_step(state: InvestigationState) -> dict:
    tracker = get_tracker()
    tracker.start("my_step", "Doing something")
    # ... read from state, do work ...
    tracker.complete("my_step", fields_updated=["evidence"], message="Done")
    return {"field": value}  # partial state update dict
```

## Rules

- Nodes receive full state, return a **partial dict** of fields to update
- Always edit `routing.py` alongside `graph_pipeline.py` when adding conditional edges
- New state keys go in `AgentState` (TypedDict) AND `AgentStateModel` (Pydantic) in `state.py`
- Use `@traceable(name="node_xxx")` for LangSmith tracing on all node functions
- Use `get_tracker()` for CLI progress output
- Use `InvestigateInput.from_state(state)` in investigation nodes to extract typed inputs
- The investigation loop is capped at 5 total iterations (see `should_continue_investigation`)
- `InvestigationState` is an alias for `AgentState`

---
> Source: [Tracer-Cloud/opensre](https://github.com/Tracer-Cloud/opensre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
