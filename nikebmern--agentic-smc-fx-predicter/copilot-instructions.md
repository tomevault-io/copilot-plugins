## smartflow-production

> SmartFlow AI production safety and deployment rules


# SmartFlow AI Production Rules

- Treat `render.yaml` as the production service topology and Render environment variables as the source of truth.
- Never create, overwrite, print, or commit production credentials. Keep `.env` local and use `.env.example` only for safe placeholders.
- Preserve the separated API, AI worker, scheduler, Telegram worker, and Redis services.
- Run `python run.py migrate` for database bootstrap and schema upgrades; do not use bare `alembic upgrade head` on a new database.
- Keep the deterministic SMC/ICT rule engine authoritative. ML may adjust confidence but must not override rule-engine trade decisions or risk controls.
- Before deployment, run the Python tests and both frontend production builds.
- Do not mark a release ready when migrations, health checks, workers, Telegram polling, or frontend builds fail.
- Never deploy from an unreviewed working tree or bypass required GitHub checks.

---
> Source: [NikebMERN/Agentic-SMC-FX-Predicter](https://github.com/NikebMERN/Agentic-SMC-FX-Predicter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
