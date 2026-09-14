## zbx

> This project is a small FastAPI service that displays host and group information

# Repository Guidelines

This project is a small FastAPI service that displays host and group information
from a Zabbix instance. The API logic is in `app/zabbix_api.py` and the web
pages are rendered with Jinja2 templates under `templates/`.

## Development
- Use **Python 3** with [PEP 8](https://peps.python.org/pep-0008/) style.
- Keep imports at the top of files and remove unused imports.
- Write functions with clear names and add comments or docstrings when needed.
- All environment values (e.g. `ZABBIX_URL`, `ZABBIX_USER`, `ZABBIX_PASSWORD`)
  are loaded with `python-dotenv`. Do not hard‑code secrets.
- Functions in `app/zabbix_api.py` accept an optional `name_filter` argument to
  filter hosts by **substring** in addition to group membership.
- Use `uvicorn main:app --reload` to run the application during development.
- There are no automated tests. Do **not** attempt to run tests.

## MikroTik Integration
- MikroTik devices are accessed using the `librouteros` library.
- Configure `MIKROTIK_USER` and `MIKROTIK_PASSWORD` in the environment.
- The `/metrics` endpoint exposes channel status and ICMP statistics for
  hosts in the `xfit` group. Channel information is taken from hosts named
  `<name>.Gr3`. It also exports `ICMP response time avg 1m` for hosts in the
  `xfit_reserve` group.
- If the MikroTik host is `ALT OF.Gr3` or `SEL.Gr3`, ICMP metrics are read from
  the Zabbix hosts `ALT OF` and `SEL` respectively.  The channel is considered
  `main` when these hosts respond to ping and `unknown` otherwise.
- Special channels map `main` to ``1`` and `unknown` to ``0``.  Other hosts use
  ``1`` for `main`, ``0`` for `backup`, and ``-1`` for `unknown`.

- The `/icmp_stats` endpoint returns the same metrics as JSON for Grafana.

## Commit Messages
Describe the intent of the change briefly, e.g. "Add error handling to
zabbix_request".

## Pull Request Notes
Include a short summary of the change and mention any manual steps required.

## Logging
Use print statements or a simple logging setup for debugging. Log files may be stored as `log.txt` when running the server with `nohup`.

## Docker Compose
The `docker-compose.yml` file launches the FastAPI service along with
Prometheus and Grafana. Define your Zabbix and MikroTik credentials in a
`.env` file and run:

```
docker compose up --build
```

Prometheus listens on port `9090`, Grafana on `3000` (admin password `admin`),
and the API on port `8000`.

---
> Source: [nitro807/zbx](https://github.com/nitro807/zbx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
