## dashboard-saedas

> Core application code lives in `app/`.

# Repository Guidelines

## Project Structure & Module Organization
Core application code lives in `app/`.
- `app/app.py`: Streamlit entrypoint and page routing.
- `app/app_pages/`: feature pages (`home.py`, `consulta.py`, `exame.py`, etc.).
- `app/components/`: shared UI pieces (filters, footer).
- `app/utils/`: data loading, schemas, style helpers.
- `app/data/`: CSV datasets consumed by dashboards.
- `app/assets/`: static files (CSS, logo, favicon).

Operational scripts include `app/exportar_dados_csv.sh`, `app/fechamento_saedas.sh`, and `gerar_relatorio_pdf.py` (report generation).

## Build, Test, and Development Commands
- `python -m venv .venv` then `.\.venv\Scripts\activate` (Windows): create/activate local env.
- `pip install -r requirements.txt`: install dependencies.
- `streamlit run app/app.py`: run app locally with hot reload.
- `docker compose up --build -d`: build and start containerized app on port `8501`.
- `docker compose logs -f streamlit`: follow runtime logs.
- `docker compose down`: stop services.

## Coding Style & Naming Conventions
Use Python 3.12+ conventions with 4-space indentation and PEP 8 naming.
- Functions/modules/variables: `snake_case`.
- Classes: `PascalCase`.
- Keep page handlers named `page_<feature>` (e.g., `page_vacinacao`).
- Prefer small, composable helpers in `app/utils/` and reusable UI in `app/components/`.
- Keep imports grouped: standard library, third-party, local.

No formatter/linter is configured yet; keep code style consistent with existing files and avoid unrelated reformatting.

## Testing Guidelines
There is currently no automated test suite in this repo. For changes:
- Run `streamlit run app/app.py` and verify affected pages manually.
- Validate deep links, e.g. `/?menu=Aluno&aluno=Nome&nasc=2010-01-01`.
- For data logic changes, add focused `pytest` tests under a new `tests/` folder using `test_*.py` naming.

## Commit & Pull Request Guidelines
Follow concise commit messages; current history aligns best with Conventional Commits:
- `feat: ...`, `fix: ...`, `docs: ...`, `chore: ...`.

For PRs, include:
- Clear summary and motivation.
- Affected paths/modules.
- Manual validation steps (and screenshots for UI changes).
- Linked issue/ticket when available.

## Security & Configuration Tips
Do not commit secrets or production credentials. Move DB/server credentials used by shell scripts to environment variables or a local, untracked config. Treat CSV files as sensitive health/student data and sanitize before sharing externally.

---
> Source: [marcelodarckferreira/Dashboard-SAEDAS](https://github.com/marcelodarckferreira/Dashboard-SAEDAS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
