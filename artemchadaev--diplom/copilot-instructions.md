## diplom

> > **FOR AI AGENTS & DEVELOPERS:** This file is your absolute starting point. It acts as the master directory. Start here to navigate the project, understand boundaries, and find technical specs. Do **not** guess rules; follow the links below.

# Diplom — Pharmaceutical ERP System (AI Context Map)

> **FOR AI AGENTS & DEVELOPERS:** This file is your absolute starting point. It acts as the master directory. Start here to navigate the project, understand boundaries, and find technical specs. Do **not** guess rules; follow the links below.

A full-stack ERP system for pharmaceutical warehouse management, covering drug batch lifecycle from inbound receiving to order shipment. Built with a **Go** backend (clean/onion architecture) and a **Next.js 16** frontend.

---

## 🗺️ Master Navigation Map

If you need to write code, design a component, or understand logic, you **MUST** read the specific README/SOP for that domain first.

### 🖥️ Frontend Sector
Where to look if you are writing React components, forms, hooks, or styles.
* **[Frontend Directives (Read First!)](./docs/frontend/README.md)** — The entry point. Explains rule separations and gives you links to:
  * **[Conventions](./docs/frontend/conventions.md)** (Styling, Tailwind v4 variables, HTTP)
  * **[Forms Logic](./docs/frontend/forms-logic.md)** (API calls, Zod schemas, validation)
  * **[Forms Specs](./docs/frontend/forms/)** (shadcn UI guidelines per form)
  * **[Hooks rules](./docs/frontend/hooks.md)** (Where all frontend business logic lives)

### 🔧 Backend Sector
Where to look if you are modifying handlers, services, repositories, or DB migrations.
* **[Backend Directives (Read First!)](./docs/backend/README.md)** — The entry point. It commands you to read:
  * **[SOP (Standard Operating Procedure)](./docs/backend/sop.md)** — The absolute canon for Onion Architecture and error handling.
  * **[Logging Rules](./docs/backend/logging.md)** — Strict rules (`fmt` is forbidden).
  * **[Migrations](./docs/backend/migrations.md)** / **[Domain logic](./docs/backend/domain-repository-service.md)**

### 📡 API Contracts
* **[API Specs & Swagger](./docs/api/README.md)** (or `docs/api/swagger.json`)
  > **Single Source of Truth:** When Backend and Frontend disagree, the Swagger JSON / API documentation is the final authority.

### ⚙️ Architecture & Deep Dives
* **Business algorithms (FEFO, etc.)** → [`docs/Diplom/algotirm.md`](./docs/Diplom/algotirm.md)
* **High-Level Entity Architecture** → [`docs/Diplom/architecture.md`](./docs/Diplom/architecture.md)
* **Database schema (full SQL text)** → [`docs/Diplom/migrate.md`](./docs/Diplom/migrate.md)
* **Git Workflows (Merge-All, etc.)** → [`docs/other/git-workflow.md`](./docs/other/git-workflow.md)

---

## ⛔ Branching & Commits Rules

| Branch | Rule |
|--------|---------|
| `main` | Production-ready code only. **NO DIRECT COMMITS.** |
| `develop-backend` | **ALL** backend feature development goes here. |
| `develop-frontend` | **ALL** frontend feature development goes here. |

Merging into `main` is handled strictly by the automated [`/merge-all`](./docs/other/git-workflow.md) workflow.

---

## 🚀 Quick Start (Local Setup)

### Prerequisites
- Docker & Docker Compose
- Copy `.env.example` → `.env` and fill in required values

### Run Everything
```bash
# Start all services (PostgreSQL, Valkey, Backend, Frontend) + run migrations
docker compose --profile all up -d
```

### Backend Only (for backend development)
```bash
docker compose --profile backend-all up -d
docker compose logs -f backend
```

### Frontend Only (for UI development)
```bash
cd frontend
npm install
npm run dev    # http://localhost:3000
```

---
> Source: [ArtemChadaev/Diplom](https://github.com/ArtemChadaev/Diplom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
