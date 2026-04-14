## crawlers

> - Python app marketplace crawler: `app_spider/` (Scrapy project). Spiders in `app_spider/app_spider/spiders/`, pipelines/config in `app_spider/app_spider/`, entry scripts under `app_spider/process/`.

# Repository Guidelines

## Project Structure & Module Organization
- Python app marketplace crawler: `app_spider/` (Scrapy project). Spiders in `app_spider/app_spider/spiders/`, pipelines/config in `app_spider/app_spider/`, entry scripts under `app_spider/process/`.
- Go Maimai crawler: `maimai_spider/` with `cmd/`, `fetchers/`, `models/`, `utils/`, plus Go modules (`go.mod`, `go.sum`).
- CEX utilities: `cex/` (Python scripts, `Dockerfile`, `docker-compose.yaml`, `supervisord.conf`). Logs/JSON persist via `cex/logs/`, `cex/json/`.
- Other examples/tools: `others/` (e.g., Django sample `others/douban_group_spy/`).
- Top-level docs: `README.md`, this `AGENTS.md`.

## Build, Test, and Development Commands
- Python (Scrapy): `cd app_spider && pip install -r requirements.txt && scrapy list` — list spiders. Run: `scrapy crawl qimai -O out.json`.
- Python scripts (CEX): `python cex/okx-network-status.py`. Docker: `cd cex && docker compose up -d --build`.
- Go: `cd maimai_spider && go build ./...` (build), `go run ./...` (run), `go mod tidy` (deps).
- Django sample: `cd others/douban_group_spy && make run` or `python manage.py runserver`.

## Coding Style & Naming Conventions
- Python: 4-space indent; snake_case for files/functions/variables; PascalCase for classes; prefer type hints. Use `ruff`/`flake8` if available.
- Go: follow `gofmt`/`go vet`; package names lower-case; exported names CamelCase.
- Keep modules small and focused; avoid unrelated changes in a single PR.

## Testing Guidelines
- Python (Scrapy): tests under `app_spider/tests/` named `test_*.py`. Run with `pytest`.
- Go: add `*_test.go` next to code. Run `go test ./...`.
- Aim for meaningful coverage on fetchers/parsers; keep tests deterministic.

## Commit & Pull Request Guidelines
- Commits: imperative mood with optional scope. Examples: `feat(app_spider): add vivo spider`, `fix(cex): handle network timeout`, `refactor(maimai_spider/utils): simplify logger`.
- PRs: concise description, reproduction steps (if bug), sample outputs/screenshots, linked issues. Ensure `go build ./...` and primary Python scripts run locally.

## Security & Configuration Tips
- Do not commit secrets/cookies. Use env vars (e.g., `export API_KEY=...`). For CEX scripts set `LARK_WEBHOOK_URL` before running (docker-compose passes it through).
- Respect target sites’ robots/ToS; tune `DOWNLOAD_DELAY` and concurrency in `app_spider/app_spider/settings.py`.
- Networked scripts should include timeouts/retries; handle API failures gracefully.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dubuqingfeng) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
