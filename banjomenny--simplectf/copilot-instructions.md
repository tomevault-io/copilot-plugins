## simplectf

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Layout

```
ctf/
├── README.md
├── .gitignore
├── setup.sh          ← Interactive setup wizard (pulls images, writes manager/.env)
├── challenge/        ← BankingAI CTF (PHP + MySQL, 5 flags)
│   ├── docker-compose.yaml
│   ├── web/          ← PHP 8.2 Apache container
│   │   ├── Dockerfile
│   │   └── src/      ← Web root (live-mounted into container)
│   ├── db/           ← MySQL 8.0 init scripts
│   │   ├── bankingai.sql       ← Clean MySQL 8.0 schema + seed data
│   │   └── init_flags.sh       ← Injects FLAG_SQL_INJECTION into users table at DB init
│   └── scripts/      ← Manual multi-team bash helpers
├── task-1/           ← SWOCTS Task 1 (Python Flask, 3 flags, no MySQL)
│   ├── docker-compose.yml
│   ├── Dockerfile
│   ├── app.py
│   ├── generate_artifacts.py
│   ├── requirements.txt
│   ├── static/
│   └── templates/
├── manager/          ← Flask web app for team registration + instance management
│   ├── docker-compose.yaml
│   ├── .env.example  ← Template — copy to .env and fill in values
│   ├── Dockerfile
│   ├── app.py
│   ├── requirements.txt
│   ├── config/
│   │   ├── bankingai.json   ← Flags + hints for BankingAI CTF
│   │   └── task1.json       ← Flags + hints for SWOCTS Task 1
│   └── templates/
└── .github/
    └── workflows/
        ├── publish-bankingai.yml   ← Publishes bankingai-web:latest to GHCR
        └── publish-task1.yml       ← Publishes task1-web:latest to GHCR
```

## Running the CTF Challenge (challenge/)

```bash
cd challenge

# Start single instance (first run takes ~30s for DB to init)
docker compose up --build -d

# Stop
docker compose down

# Full reset (wipes DB volume)
docker compose down -v && docker compose up --build -d

# Multi-team: each team gets its own containers on a separate port
bash scripts/add_team.sh <name> [port]   # auto-assigns port from 8000 if omitted
bash scripts/remove_team.sh <name>       # tears down + wipes DB volume
bash scripts/list_teams.sh              # show running teams and ports
```

The single-instance default is at **http://localhost** (port 80). Multi-team instances are at the auto-assigned port. The port is controlled by the `PORT` env var in `docker-compose.yaml` (`${PORT:-80}:80`).

## Quick Setup (recommended)

```bash
# Run the interactive wizard from the repo root:
bash setup.sh
# Then:
cd manager && docker compose up -d
```

The wizard:
1. Asks which CTF to run (BankingAI or SWOCTS Task 1)
2. Asks for HOST_IP
3. Auto-generates SECRET_KEY, ADMIN_TOKEN, FLAG_SECRET
4. Pulls the pre-built image from ghcr.io and tags it locally
5. Writes `manager/.env`

## Running the Manager Manually (manager/)

```bash
# 1. Copy the example env file and fill in your values
cp manager/.env.example manager/.env
# Edit manager/.env: set HOST_IP, SECRET_KEY, ADMIN_TOKEN, FLAG_SECRET,
#   CTF_COMPOSE_HOST_PATH, CTF_CHALLENGE_DIR, CTF_CONFIG_FILE, CTF_NAME

# 2. Build the challenge image (BankingAI example)
cd challenge && docker compose build

# 3. Start manager
cd manager && docker compose up --build -d
# Browse to http://localhost
```

**Key env vars in `manager/.env`:**
| Variable | Description |
|---|---|
| `CTF_COMPOSE_HOST_PATH` | Host path to the challenge docker-compose file |
| `CTF_CHALLENGE_DIR` | Host absolute path to the challenge directory |
| `CTF_CONFIG_FILE` | Container path to the JSON config (e.g. `/ctf/config/bankingai.json`) |
| `CTF_NAME` | Display name shown in page titles |
| `WEB_SERVICE_NAME` | Docker service name of the web container (default `web`) |
| `STARTUP_TIMEOUT` | Seconds to wait for web container ready (default `180`) |

`manager/.env` is gitignored — never commit it.

## Challenge Architecture

The CTF is a PHP employee portal ("BankingAI Cloud") backed by MySQL. Only port 80 is exposed.

**Flag locations and how they are set:**
| Flag env var | Where it appears |
|---|---|
| `FLAG_INSPECTED` | HTML comment in `products.php` (view source) |
| `FLAG_LOGIN` | Shown on `dashboard.php` after login |
| `FLAG_SQL_INJECTION` | Written as a `username` in the `users` DB table by `init_flags.sh` |
| `FLAG_USER_ESCALATION` | Rendered in `admin_subnav.php` nav link |
| `FLAG_FILE_UPLOAD` | Written to `/flag.txt` at container start (via `web/Dockerfile` CMD) |

**Intended exploit chain:**
1. `robots.txt` → `/staff-resources/new-employee-guide.txt` → credentials `ajohnson:Welcome2026`
2. Login → `dashboard.php` shows `FLAG_LOGIN`
3. `lookup.php` SQL injection (unsanitised `WHERE full_name LIKE '%$search%'`) → dump `users` table → get `FLAG_SQL_INJECTION` and admin password hash
4. Login as admin → `admin_subnav.php` shows `FLAG_USER_ESCALATION`
5. `admin_uploads.php` (no file type validation) → upload PHP webshell → execute → read `/flag.txt` = `FLAG_FILE_UPLOAD`
6. `FLAG_INSPECTED` is in the HTML source of `products.php` (can be found at any point)

## Customising Flags

Edit environment variables in `challenge/docker-compose.yaml` then rebuild. The `FLAG_SQL_INJECTION` value is picked up by `db/init_flags.sh` at DB initialisation — no SQL edits needed.

## Key Implementation Notes

- `web/src/` is bind-mounted into the container, so PHP file edits take effect immediately without rebuild.
- `web/src/login.php` uses a prepared statement (intentional — login bypass is via credential discovery, not SQLi).
- `web/src/lookup.php` is intentionally vulnerable to SQL injection (the main exploitation step).
- All admin pages (`admin.php`, `admin_users.php`, `admin_logs.php`, `admin_uploads.php`) must call `session_start()` before the role check — the role check runs before `internal_theme.php` is included.
- `internal_theme.php` guards `session_start()` with `if (session_status() === PHP_SESSION_NONE)` to prevent double-call warnings.
- `db/bankingai.sql` is a clean MySQL 8.0 file (no `@OLD_*` compatibility headers from mysqldump). Do not replace it with a raw mysqldump output from MySQL 5.7.
- Admin passwords: `adm_ewright` = `P@ssw0rd99`, `adm_msmith` = `Welc0me@2` (both in rockyou.txt; MD5 hashed in SQL).
- The `uploads/` directory is gitignored and world-writable; PHP files placed there execute.

## Manager Implementation Notes

- `manager/app.py` — all routes, bcrypt hashing, SQLite DB, background thread that polls until team's web container is reachable.
- `CTF_COMPOSE_FILE` in `manager/docker-compose.yaml` must be the **container** path — the file is bind-mounted from `CTF_COMPOSE_HOST_PATH` on the host into `/ctf/challenge/docker-compose.yaml` inside the container.
- `CTF_CHALLENGE_DIR` must be the **host** filesystem path — Docker resolves relative bind mounts (e.g. `./web/src`) against the host, not the container.
- Status polling hits `http://HOST_IP:PORT` from inside the manager container, so `HOST_IP` must be a LAN IP or hostname reachable from inside Docker (not `127.0.0.1` unless testing locally with host networking).
- `manager/data/manager.db` is gitignored; it is created automatically on first run.
- CTF config is loaded from `CTF_CONFIG_FILE` (JSON) at startup. Falls back to hardcoded BankingAI defaults if unset or unreadable.
- `CTF_NAME_EFFECTIVE` is set from the JSON config (or `CTF_NAME` env var) and injected into all Jinja2 templates as `ctf_name`.
- `WEB_SERVICE_NAME` controls which Docker service is polled and started (default `web`).
- `STARTUP_TIMEOUT` controls the per-team container readiness timeout (default `180`s).

---
> Source: [Banjomenny/simpleCTF](https://github.com/Banjomenny/simpleCTF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
