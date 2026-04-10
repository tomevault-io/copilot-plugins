## smoke-alarm

> Purpose: give coding agents one predictable place to learn how to build, test, lint, type check, route, and ship work in this repo without breaking things. Keep human onboarding in README.md.

AGENTS.md

Purpose: give coding agents one predictable place to learn how to build, test, lint, type check, route, and ship work in this repo without breaking things. Keep human onboarding in README.md.

Project profile

Name: Smoke-Alarm

Language: Python 3.11 or newer

Domain: smoke alarm compliance operations in Victoria AU

Top folders

agent Python package for CLI, services, optimizer, reports

gui Dispatcher UI

docs Runbooks and architecture notes

.github/workflows CI

scripts Local helpers

templates Report templates

apps_script Google Apps Script assets

Golden rules

Rebase on origin/main before any push.

Never commit conflict markers.

Never commit secrets. Service account JSON stays outside the repo.

Keep PRs small and focused. Update tests and docs with code changes.

Always run preflight before pushing.

Environment variables for local runs and CI (do not commit secrets)
OSRM_BASE_URL=http://localhost:5000

NOMINATIM_BASE_URL=https://nominatim.openstreetmap.org

GEOCODE_USER_AGENT=SmokeAlarmBot/1.0 (contact@example.com
)
GEOCODE_RPS=1
GOOGLE_SA_PATH=.secrets/service_account.json
APPRISE_URLS=
API_TOKEN=
PYTHONPATH=agent/src

Setup (use uv if available, else pip)
uv pip install --system -U pip
uv pip install --system -r requirements.txt || true
uv pip install --system ruff mypy pytest pytest-xdist coverage pytest-cov hypothesis bandit pip-audit deptry pipdeptree pre-commit nox
pre-commit install
pre-commit install --hook-type pre-push

Optional local OSRM for Australia. Prefer this to public demo.
Put australia-latest.osm.pbf in ./data then run:
docker run -t -v $(pwd)/data:/data osrm/osrm-backend osrm-extract -p /opt/car.lua /data/australia-latest.osm.pbf
docker run -t -v $(pwd)/data:/data osrm/osrm-backend osrm-partition /data/australia-latest.osrm
docker run -t -v $(pwd)/data:/data osrm/osrm-backend osrm-customize /data/australia-latest.osrm
docker run -p 5000:5000 -t -v $(pwd)/data:/data osrm/osrm-backend osrm-routed --algorithm mld /data/australia-latest.osrm

Daily preflight
ruff check . --fix
ruff format .
mypy .
pytest -n auto --cov=agent --cov-report=term
bandit -q -r agent -x tests
pip-audit -r requirements.txt --strict || true
deptry

Git hygiene
git fetch origin
git rebase origin/main
git grep -n -E '^(<{7}|={7}|>{7}||{7})' -- . && exit 1

Stable entry points
CLI route planning:
python -m agent.cli route --date YYYY-MM-DD --responses-file path/to/jobs.csv
Optional flags if present:
--vehicles N
--shift-start HH:MM
--shift-end HH:MM
--max-visits-per-tech N
--output pdf|csv|sheet

Service rules

Geocoding uses NOMINATIM_BASE_URL and GEOCODE_USER_AGENT and obeys GEOCODE_RPS.

Routing uses OSRM_BASE_URL. Do not use the public OSRM demo in production.

Sheets uses gspread with GOOGLE_SA_PATH. Share the sheet with that SA email.

Reports render with Jinja2 templates in templates and WeasyPrint for PDF.

UI starts with: streamlit run gui/streamlit_app.py

API if enabled runs at agent/src/agent/api/main.py and uses API_TOKEN header.

Coding standards

Ruff rules from pyproject.toml

mypy strict mode

Prefer pure functions and small modules

Log with structured messages and include correlation ids where helpful

Do not modify without reason

.github/workflows steps that enforce lint, type, tests

.secrets directory and any secret files

LICENSE

Commit and PR rules

Conventional commits

PR must include what changed, why, and test evidence

PR must pass CI and be up to date with origin/main

Self checks before opening a PR
ruff check . --fix && ruff format .
mypy .
pytest -n auto --cov=agent --cov-report=term
pip-audit -r requirements.txt --strict || true
git fetch origin && git merge-base --is-ancestor origin/main HEAD
git grep -n -E '^(<{7}|={7}|>{7}||{7})' -- . || echo "No conflict markers"

Nested AGENTS.md
Add scoped AGENTS.md files inside key subfolders to refine rules for that area. The root file stays the source of truth. Keep nested files short and consistent.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Billionhair)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Billionhair)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
