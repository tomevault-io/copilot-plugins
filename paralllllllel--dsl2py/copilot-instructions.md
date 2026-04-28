## dsl2py

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**DSL2Py** is a lightweight workflow engine that converts YAML-based DSL files into executable Python workflows. It uses a decorator-based node registry and BFS execution engine.

**Version**: v0.2.0

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run workflow via CLI
python main.py template.yml

# Run via Python API
python main.py <yaml_file>  # with inputs in code

# Start API server
python app.py                    # basic
python app.py --reload           # auto-reload dev mode
python app.py --workers 4        # production

# Test API
python -m pytest                 # all tests
python -m pytest tests/          # specific test dir

# Quick smoke test (streams to main.log)
python test_simple.py
```

## Architecture

### Execution Flow
```
YAML DSL → DSLParser → WorkflowGraph → WorkflowEngine (BFS) → Result
```

### Node System
- All nodes inherit `BaseNode` (`nodes/base.py`) and auto-register via `@NodeRegistry.register('Type')`
- 6 built-in types: Start, LLM, Judge, VariableHandle, Script, End
- Adding a new node: create class in `nodes/`, decorate with `@NodeRegistry.register()`, implement `execute(context, **kwargs)`

### WorkflowContext (`core/context.py`)
- Stores `{node_id: {field: value}}` for all node outputs
- Variable pattern: `${node_id.field_name}` or `${sys.variable}`
- **Critical**: Unexecuted nodes (from unselected branches) resolve to empty string, not literal `${...}`

### WorkflowEngine (`core/engine.py` - BFS)
- **Key behavior**: Executes current node from queue, not downstream nodes
- Start node executed before BFS loop, then skipped
- End nodes skip dependency checking (may have multiple upstream branches)
- Other multi-upstream nodes: retry 3 times for dependencies, then force-execute
- Judge nodes save `target_ids` to context for routing
- Uses `queued` set to prevent duplicate enqueueing

### Judge Node (`nodes/judge.py`)
1. Evaluate conditional branches first (in order)
2. Use default branch only if no conditions match
3. **Must save `target_ids` to context** — engine reads this for routing

### LLM Integration (`utils/llm_client.py`, `nodes/llm.py`)
- Uses OpenAI SDK for DashScope/OpenAI/Azure compatibility
- Vision model detection: checks `capabilities` field for 'VU', `vlImageUrl` param, or model name containing 'vl'/'vision'
- Vision models require `content` as list with `type: image_url` objects

### Script Node (`nodes/script.py`)
- Executes user Python code via `exec()` with sandboxed globals (`json` + input params)
- Must define `main()` function returning dict
- Retry via `retryConfig`, error strategies: `noop`, `fallback`, `throw`

## Configuration

API keys from `config.ini` (priority): explicit params > env vars > config file > defaults

```ini
[dashscope]
api_key = your-key
base_url = https://dashscope.aliyuncs.com/compatible-mode/v1
model = qwen-max-latest
```

## Common Debugging Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Judge routing all branches | `target_ids` not saved to context | Add `context.set_value(self.id, 'target_ids', target_ids)` |
| `${NodeID.field}` in result | Branch not taken, returns empty string | Expected behavior — unexecuted nodes return `""` |
| End node not executing | Dependency check blocking multi-upstream | End nodes skip dependency check; others force-execute after 3 retries |
| Vision model not called | Detection only checked `capabilities` | Also checks `vlImageUrl` param and model name pattern |

## Frontend: Tailwind CSS v4 Caveats

The frontend uses **Tailwind CSS v4.2**, which has breaking changes from v3:

- **`space-y-*` / `space-x-*`**: v4 uses CSS `gap` internally, so these only work on `flex` or `grid` containers. In v3 they used `margin` and worked on any container.
- **`px-*` / `py-*` / `gap-*` / `mb-*` and other spacing utilities**: May silently fail in certain layout combinations (e.g. with `overflow-y-auto`, nested `flex`).
- **Workaround**: Use **inline styles** for critical spacing/padding/margin/gap instead of Tailwind classes to guarantee they apply:
  ```tsx
  // BAD - may not apply in v4
  <div className="px-6 py-5 space-y-7">

  // GOOD - always applies
  <div style={{ padding: '20px 28px', display: 'flex', flexDirection: 'column', gap: '24px' }}>
  ```
- Non-spacing utilities (colors, borders, border-radius, `flex`, `items-center`, etc.) are generally unaffected and can continue using Tailwind classes.

## API Endpoints

- `POST /workflow/run` — async execution
- `POST /workflow/run-sync` — sync execution
- `GET /task/{task_id}` — query status
- `GET /workflows` — list available workflows
- `GET /health` — health check
- Swagger docs at `http://localhost:8000/docs`

## Project Structure

```
dsl2py/
├── core/              # engine: parser, registry, workflow, engine, context
├── nodes/             # node implementations: start, llm, judge, variable, script, end
├── api/               # FastAPI service with routes/
├── utils/             # llm_client, log_config
├── main.py            # CLI entry + DSL2Py.quick_run()
├── app.py             # API server launcher
└── template.yml        # sample workflow
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/paralllllllel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
