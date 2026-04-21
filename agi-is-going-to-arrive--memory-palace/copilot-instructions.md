## memory-palace

> These rules apply to the `Memory-Palace/` repository.

# Memory Palace Agent Rules

These rules apply to the `Memory-Palace/` repository.

## Read First

- `README.md`
- `docs/GETTING_STARTED.md`
- `docs/skills/memory-palace/SKILL.md` when the task is about durable memory workflow or skill behavior

## Durable Memory Workflow

- Start with `read_memory("system://boot")` before the first real Memory Palace operation in a session.
- If the target URI is unknown, use `search_memory(..., include_session=true)` before guessing a path.
- Before `create_memory`, `update_memory`, `delete_memory`, or `add_alias`, read the target or the best matching candidate first.
- If `guard_action` is `NOOP`, stop the write, inspect `guard_target_uri` / `guard_target_id`, read the suggested target, then decide whether anything should change.
- Prefer repo-visible references under `docs/skills/memory-palace/` instead of hidden mirror-relative paths.

## Code And Validation

- Keep changes scoped to this repository.
- Backend validation: `cd backend && .venv/bin/pytest tests -q`
- Frontend validation: `cd frontend && npm test && npm run build`
- Skill validation: `python scripts/evaluate_memory_palace_skill.py`
- Live MCP validation: `cd backend && python ../scripts/evaluate_memory_palace_mcp_e2e.py`

## MCP Binding

- Use the repo-local MCP binding that matches the host:
- native Windows: `backend/mcp_wrapper.py`
- macOS / Linux / `Git Bash` / `WSL`: `scripts/run_memory_palace_mcp_stdio.sh`

---
> Source: [AGI-is-going-to-arrive/Memory-Palace](https://github.com/AGI-is-going-to-arrive/Memory-Palace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
