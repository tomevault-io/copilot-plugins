## minibench

> Setup, run, and component details live in `README.md` and `agentbench/README.md`.

# MiniBench agent context

Setup, run, and component details live in `README.md` and `agentbench/README.md`.

## Environment

- `./scripts/setup-dev.sh` creates the three Python virtualenvs and installs frontend dependencies.
- The FastAPI backend runs on port 3070. Vite runs on 5173 and proxies `/api` and `/health` to 3070.
- PostgreSQL must be reachable for the backend. The documented Docker mapping is host port 5438; the documented local-Postgres example uses 5432.
- Run `agentbench` modules from the repository root. `--dry-run` is offline; live OpenRouter runs require `OPENROUTER_API_KEY` from the root `.env`.
- `minibench run` requires an Ollama daemon. `--publish` writes results to the configured backend.

## Verification

These are the repository's CI-equivalent checks:

```bash
cd backend && pytest
cd agentbench && pytest -q
cd cli && pytest
cd frontend && npm run lint && npm test && npm run build
python -m agentbench.run --config agentbench/presets/moa-v1.yaml --tasks agentbench/tasks/coding-v1.json --trials 2 --dry-run --out /tmp/dryrun.json
```

Backend tests require PostgreSQL; set `MINIBENCH_TEST_PG_HOST` and `MINIBENCH_TEST_PG_PORT` when the listener is not the documented Docker default.

## Hard invariants

- Never commit credentials or represent dry-run or synthetic benchmark data as live results.
- Paid live model calls, publishing, deployment, and external communication require explicit owner authorization.

<!-- unhobbled 2026-07-28; re-ablate after 2027-01-28 -->

---
> Source: [RaapTechllc/minibench](https://github.com/RaapTechllc/minibench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
