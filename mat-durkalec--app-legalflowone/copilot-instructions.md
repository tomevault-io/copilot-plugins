## app-legalflowone

> Marketing website + REST API backend for **LegalFlow One**, a company that helps businesses establish and manage their companies. Includes a full-stack Docker deployment with CI/CD to an OVH VPS.

# LegalFlow One — Project Reference

## What This Project Is

Marketing website + REST API backend for **LegalFlow One**, a company that helps businesses establish and manage their companies. Includes a full-stack Docker deployment with CI/CD to an OVH VPS.

---

## Project Structure

```
app.legalflowone/
├── Dockerfile                  # Python backend image (builds from src/)
├── docker-compose.yml          # Orchestrates nginx, frontend, backend, db
├── .env                        # Local secrets (never commit)
├── .env.example                # Template for .env
├── nginx/
│   └── nginx.conf              # Proxies /* to frontend, /api/* to backend
├── frontend/                   # Next.js 15 marketing site
│   ├── Dockerfile              # Multi-stage Node 20 build (standalone output)
│   ├── next.config.mjs         # Next.js config (must be .mjs, not .ts)
│   ├── tailwind.config.ts
│   ├── postcss.config.js
│   ├── package.json
│   ├── public/                 # Must exist for Docker build (keep .gitkeep if empty)
│   └── src/
│       ├── app/                # Next.js App Router (layout.tsx, page.tsx, globals.css)
│       ├── components/         # One file per page section (13 components)
│       └── lib/
│           └── data.ts         # All website copy and data arrays — edit here
├── src/
│   ├── app.py                  # Flask API
│   └── requirements.txt        # Python dependencies
├── db/
│   └── init.sql                # MySQL schema + seed data
└── .github/
    └── workflows/
        └── deploy.yml          # GitHub Actions CI/CD pipeline
```

---

## Stack

| Layer    | Technology                          |
|----------|-------------------------------------|
| Frontend | Next.js 15 + TypeScript + Tailwind  |
| Backend  | Python 3.12 + Flask                 |
| Database | MySQL 8.0                           |
| Server   | NGINX (alpine)                      |
| Runtime  | Docker + Docker Compose             |
| CI/CD    | GitHub Actions                      |

---

## Services (docker-compose)

| Service    | Image / Build       | Port              | Role                              |
|------------|---------------------|-------------------|-----------------------------------|
| `nginx`    | nginx:alpine        | `80` (public)     | Reverse proxy                     |
| `frontend` | frontend/Dockerfile | `3000` (internal) | Next.js marketing site            |
| `backend`  | Dockerfile          | `5000` (internal) | Flask REST API                    |
| `db`       | mysql:8.0           | `3306` (internal) | MySQL database                    |

NGINX routes:
- `/*` → proxies to `frontend:3000` (Next.js)
- `/api/*` → proxies to `backend:5000/api/`

---

## API Endpoints

| Method | Path           | Description              |
|--------|----------------|--------------------------|
| GET    | `/api/health`  | DB connectivity check    |
| GET    | `/api/services`| Returns services table   |

---

## Database

- Database name: `legalflowone`
- App user: `legalflow`
- Initialized from `db/init.sql` on first run
- Data persisted in Docker volume `mysql_data`
- `services` table seeded with 6 company service records

---

## Environment Variables

Stored in `.env` (copy from `.env.example`):

```
DB_NAME=legalflowone
DB_USER=legalflow
DB_PASSWORD=<secret>
MYSQL_ROOT_PASSWORD=<secret>
```

---

## Running Locally

```bash
cp .env.example .env      # fill in passwords
docker compose up --build
```

- Frontend: http://localhost
- Health check: http://localhost/api/health

Stop: `docker compose down`
Logs: `docker compose logs -f frontend` / `docker compose logs -f backend`

> **Note:** `frontend/public/` must exist for the Next.js standalone Docker build to succeed. If the directory is empty, keep a `.gitkeep` inside it.

---

## Deployment — OVH VPS

- **Host:** `vps-bc406ee6.vps.ovh.net`
- **User:** `ubuntu`
- **App directory:** `/home/ubuntu/app.legalflowone`
- **Docker & Docker Compose:** already installed on the VPS

### CI/CD Flow (GitHub Actions)

Triggered on every push to `main`:
1. Copies project files to VPS via SCP (excludes `.git`, `.env`)
2. SSHs in, writes `.env` from GitHub secrets
3. Runs `docker compose up --build -d`
4. Prunes unused Docker images

### Required GitHub Secrets

| Secret               | Description                  |
|----------------------|------------------------------|
| `VPS_PASSWORD`       | SSH password for ubuntu user |
| `DB_PASSWORD`        | MySQL app user password      |
| `MYSQL_ROOT_PASSWORD`| MySQL root password          |

Add at: `GitHub repo → Settings → Secrets and variables → Actions`

---

## Frontend — Marketing Site

Next.js 15 app at `frontend/`. Page sections (in order):

- Navigation (sticky, transparent → white on scroll, mobile drawer)
- Hero (headline, two CTAs, animated product dashboard mockup)
- Problem (4 pain-point cards)
- Solution (sticky copy + 6 feature blocks)
- Features (6-card grid with hover effects)
- How It Works (3-step timeline)
- Use Cases (5 audience cards)
- Benefits (split layout)
- Differentiation (dark comparison table — no competitor names)
- Trust (4 trust-signal cards)
- CTA banner
- Footer

**Edit all copy** in `frontend/src/lib/data.ts` — titles, descriptions, and data arrays for every section are centralised there.

Color palette: White/slate base, Indigo `#4f46e5` accent. Uses `lucide-react` for icons and `framer-motion` for scroll animations.

---
> Source: [mat-durkalec/app.legalflowone](https://github.com/mat-durkalec/app.legalflowone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
