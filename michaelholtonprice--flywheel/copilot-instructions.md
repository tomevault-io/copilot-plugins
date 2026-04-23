## flywheel

> Python orchestration framework for measurable AI improvement loops. Wires Docker containers together, tracks artifacts, and provides visibility into hybrid human+agent workflows.

# Flywheel

Python orchestration framework for measurable AI improvement loops. Wires Docker containers together, tracks artifacts, and provides visibility into hybrid human+agent workflows.

## Key concepts

- **Block execution** is the atomic operation. One block, one Docker container, one set of inputs producing outputs. See `docs/vision.md` (aspirational) and `docs/architecture.md` (implemented or agreed-upon decisions; future work is noted separately at the end).
- **All project logic runs in Docker containers.** Flywheel on the host is pure orchestration.
- **Artifacts** have two levels: declarations (template-level types like "checkpoint" or "score") and instances (concrete, immutable records produced by block executions). Two storage kinds exist: copy artifacts (files in the workspace) and git artifacts (references to version-controlled code).
- **Workspaces** are created from templates and accumulate artifact instances and execution records over their lifetime. The workspace is a history, not a snapshot.
- **The foundry** is the flywheel-managed directory within a project, holding templates and workspaces. It is a peer to the project source code.

## Batteries (solve-once capabilities)

- Computer use agent (CUA) — X11 session management, screenshot loop
- Claude Code / Codex agent wrappers — container launch, auth, MCP tool exposure

## Build and test

```bash
pip install -e ".[dev]"
bash scripts/verify.sh
```

Run `scripts/verify.sh` before considering any work complete. It runs ruff and pytest.

Before committing, also run a review agent to check for:
- Test quality (coverage gaps, edge cases, phantom tests)
- Docstring completeness (Args/Returns/Raises sections for public functions with parameters — ruff does not enforce this)

## Project layout

Flywheel expects a `flywheel.yaml` in the target project root defining `foundry_dir` (the foundry directory). Templates and workspaces live under `foundry_dir`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MichaelHoltonPrice) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
