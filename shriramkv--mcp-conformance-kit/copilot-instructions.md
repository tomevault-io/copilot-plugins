## mcp-conformance-kit

> Guidance for coding agents working in this repository.

# AGENTS.md

Guidance for coding agents working in this repository.

## What this project is

`mcp-conformance-kit` is a specification conformance suite for Model Context Protocol servers. It connects to a server, exercises the protocol, and reports where the server deviates from the spec.

## Layout

- `mcpck/transport.py` raw JSON-RPC transports (stdio, streamable HTTP)
- `mcpck/core.py` result model, severity levels, check registry, run loop
- `mcpck/checks/` one module per group, each check registered with `@check`
- `mcpck/report.py` terminal, JSON, markdown and badge renderers
- `mcpck/cli.py` argparse entry point exposed as `mcpck`
- `examples/toy_server.py` compliant and deliberately broken fixture servers
- `tests/` pytest suite that runs the whole kit against both fixtures

## Setup

```bash
pip install -e ".[dev,http]"
```

## Checks before you commit

```bash
python -m pytest -q
python -m ruff check .
mcpck run "python examples/toy_server.py good"   # must exit 0
mcpck run "python examples/toy_server.py bad"    # must exit 1
```

## House rules

- Do not add a runtime dependency to the stdio path. Zero dependencies is a design constraint, not an accident. Optional extras are fine.
- Do not use an MCP SDK client anywhere in `mcpck/`. SDKs normalise the exact behaviour under test. Speak raw JSON-RPC.
- Every check needs a stable id, a `spec_ref`, and a severity. MUST means a hard spec requirement, SHOULD a recommendation, MAY an interoperability nicety.
- Checks are capability-gated. If the server never declared the capability, skip rather than fail.
- Checks are non-destructive by default. Anything that invokes a live tool must set `destructive=True` and gate on `ctx.cache["allow_invoke"]`.
- In a check written as a generator, emit skips with `yield skip(...)` followed by `return`. A bare `return skip(...)` inside a generator is discarded and silently reports a pass.
- Run order comes from `GROUP_ORDER` in `mcpck/core.py`, not from import order. Lifecycle first, hygiene last, because hygiene judges everything printed during the rest of the run. If you add a group, add it to `GROUP_ORDER`.
- Never renumber an existing check id. Ids appear in other people's CI configs and reports.
- When you add a check, extend the fixtures so the good server passes it and, where it makes sense, the bad server fails it. Then assert on it in `tests/test_suite.py`.

## Style

- Line length 100, ruff with `E, F, I, UP, B`.
- Failure messages are written for the server author, not for us. Say what the server did and why it matters, not just which assertion tripped.

---
> Source: [shriramkv/mcp-conformance-kit](https://github.com/shriramkv/mcp-conformance-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
