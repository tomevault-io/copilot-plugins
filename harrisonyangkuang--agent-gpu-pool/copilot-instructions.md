## agent-gpu-pool

> This file concerns the compute interface. Existing human authorization rules remain in force.

# Agent GPU Pool integration

This file concerns the compute interface. Existing human authorization rules remain in force.

1. Query `pool_overview` and `list_ready_results(project=...)` before creating new experiments.
2. Inspect the current baseline and write a clear experiment ID, hypothesis, metric, runtime estimate, and stopping condition.
3. Use the same registered project workspace and explicit `include` list. Keep credentials out of job specs, arguments, source code, artifacts, and conversation.
4. Submit via `submit_job` with an idempotency key. Existing admin project policy determines whether real execution is authorized. Do not change policy or add credentials on behalf of an agent without human authorization.
5. Record the returned job ID in project notes. Do not ask the user which GPU/account to use when the broker can choose.
6. Submit returns quickly. Preserve the ID, continue independent useful work, and avoid repeatedly submitting because the first call timed out.
7. Later sessions recover with `get_job` / `list_ready_results`. `LOST` means reconcile the existing run; it does not authorize another run.
8. Check both terminal status and `artifacts_ready`. Fetch metadata first, then metrics and a bounded log tail. Validate scientific results independently.
9. Compare to the current baseline; use `record_experiment_result` to save keep/reject/inconclusive and the evidence-grounded conclusion.
10. Only design another experiment when justified. New uploads, paid compute, expanded budgets, retries, GPU runs, and leaderboard submissions require their existing authorization.

Use the single `agent-gpu-pool` MCP server shared with Claude Code. No credential administration tool is exposed. `submission.csv` is an artifact, not authorization for a leaderboard submission.

Repository development: Python 3.11+, `uv sync --extra dev`, `uv run pytest -q`, `uv run ruff check src tests`. Tests use FakeWorker/mocks and tiny explicitly scoped local programs only. Never use real Kaggle credentials, GPU quota, or paid providers in tests.

Account-aware hardware: inspect `list_workers.hardware_options`. Use `required_accelerators` for a mandatory model, `min_gpu_count` for device count and `min_vram_gb` for per-device memory. Never assume all accounts have identical GPU permissions or sum two cards into one memory budget. Let the broker preserve advanced-capability accounts for jobs that need them. Hardware configuration examples are documented in `docs/account-hardware.md`.

---
> Source: [HarrisonYangKuang/agent-gpu-pool](https://github.com/HarrisonYangKuang/agent-gpu-pool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
