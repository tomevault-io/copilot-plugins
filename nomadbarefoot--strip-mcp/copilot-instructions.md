## strip-mcp

> - For repository restructuring and layout reviews, the user has requested parallel `artificer` (architecture) and `crucible` (contributor-facing clarity) subagent reviews alongside primary research.

## Learned User Preferences

- For repository restructuring and layout reviews, the user has requested parallel `artificer` (architecture) and `crucible` (contributor-facing clarity) subagent reviews alongside primary research.

## Learned Workspace Facts

- `toolgate` is a Python package (Hatchling, `src/toolgate/`) implementing MCP middleware with staged tool discovery and a `toolgate` CLI (including macOS setup helpers).
- `discover_node_mcp_servers(project_root)` reads `package.json` and registers known npm MCP packages whose entry files exist under `node_modules`; defaults live in `node_discovery.py` and callers can extend behavior via `registry=`.
- `examples/toolgate_vs_full_benchmark.py` runs the same day-to-day scenarios in toolgate (`staged=True`) versus full-schema (`staged=False`) modes and writes `examples/toolgate_vs_full_benchmark_report.json`; `examples/multi_mcp_agent_benchmark.py` uses the same discovery path.
- A root `package.json` supports optional Node MCP and example tooling; the Python library does not require Node for core functionality.
- Long-form planning and benchmark notes live under `docs/`; keep README links aligned when moving files.
- Default npm package → server mappings are exposed as `DEFAULT_NODE_MCP_REGISTRY` in `node_discovery` (re-exported from `toolgate`); `setup.discovery` uses the same public mapping.

---
> Source: [nomadbarefoot/strip-mcp](https://github.com/nomadbarefoot/strip-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
