## jpuns-claude-6-5-0

> **UNS-ClaudeJP** is a comprehensive Human Resources Management System (HRMS) tailored for Japanese staffing agencies (*Jinzai Haken*). It manages the full lifecycle of temporary workers, from candidate registration (Resume/Rirekisho) to dispatching, time tracking, and automated payroll calculation.

# UNS-ClaudeJP 6.5.0 - Gemini Context

## 1. Project Overview
**UNS-ClaudeJP** is a comprehensive Human Resources Management System (HRMS) tailored for Japanese staffing agencies (*Jinzai Haken*). It manages the full lifecycle of temporary workers, from candidate registration (Resume/Rirekisho) to dispatching, time tracking, and automated payroll calculation.

*   **Type:** Full-Stack Web Application (Monorepo structure).
*   **Primary Language:** TypeScript (Frontend), Python (Backend).
*   **Domain:** HR Tech, Staffing, Japan (requires handling Japanese text, dates, and currency).
*   **Status:** Version 6.5.0 (Active Development/Maintenance).

## 2. Architecture & Tech Stack

The system follows a containerized microservices architecture orchestrated via Docker Compose.

### Frontend (`/frontend`)
*   **Framework:** Next.js 16 (App Router).
*   **UI Library:** React 19, Tailwind CSS 3.4, Shadcn UI.
*   **State Management:** Zustand, TanStack Query v5.
*   **Testing:** Vitest (Unit), Playwright (E2E).
*   **Build Tool:** Turbopack.

### Backend (`/backend`)
*   **Framework:** FastAPI 0.115.6.
*   **Language:** Python 3.11+.
*   **Database ORM:** SQLAlchemy 2.0 (Async) + Alembic for migrations.
*   **Authentication:** JWT (HS256) with Role-Based Access Control (RBAC).
*   **OCR:** Hybrid approach (Azure Computer Vision, EasyOCR, Tesseract) for digitizing Japanese resumes.

### Infrastructure & Services
*   **Database:** PostgreSQL 15 (Service: `db`).
*   **Cache/Queue:** Redis 7 (Service: `redis`).
*   **Management:** Adminer (Service: `adminer`).
*   **Orchestration:** Docker Compose (6 services: `db`, `redis`, `backend`, `frontend`, `adminer`, `otel-collector`*).
    *   *Note: Observability stack (Prometheus/Grafana/Tempo) is present in configuration.*

## 3. Key Operational Commands

The project relies heavily on **Windows Batch Scripts** in the `scripts/` directory for daily operations. **Always prefer these scripts over raw docker commands when possible.**

### 🪟 Windows (Primary Workflow)

| Goal | Command | Description |
| :--- | :--- | :--- |
| **Start System** | `scripts\START.bat` | Starts all Docker services correctly. |
| **Stop System** | `scripts\STOP.bat` | Stops all services. |
| **View Logs** | `scripts\LOGS.bat` | Interactive menu to view service logs. |
| **Reinstall** | `scripts\REINSTALAR.bat` | **CAUTION:** Wipes data and reinstalls everything. |
| **Backup DB** | `scripts\BACKUP_DATOS.bat` | Creates a PostgreSQL dump. |
| **Diagnostics** | `scripts\DIAGNOSTICO_FUN.bat` | Runs comprehensive system health checks. |

### 🐧 Linux/Raw Docker (Alternative)

*   `docker compose up -d` - Start services.
*   `docker compose logs -f [service]` - View logs (backend, frontend, db).
*   `docker exec -it uns-claudejp-backend alembic upgrade head` - Run DB migrations manually.

### 💻 Frontend Development (`/frontend`)

*   `npm run dev` - Start dev server (Port 3000/3200).
*   `npm run lint` - Run ESLint.
*   `npm run typecheck` - Run TypeScript compiler check.
*   `npm test` - Run Unit tests (Vitest).
*   `npm run test:e2e` - Run E2E tests (Playwright).

### 🐍 Backend Development (`/backend`)

*   `pytest` - Run all tests.
*   `alembic revision --autogenerate -m "msg"` - Create migration.
*   `alembic upgrade head` - Apply migrations.
*   `python scripts/import_data.py` - Import initial employee data.

## 4. Development Conventions & Rules

Derived from `CLAUDE.md` and `.claude/CLAUDE.md`.

1.  **Strict File Protection:**
    *   **NEVER** modify `scripts/*.bat`, `docker-compose.yml`, or `.env` without explicit user confirmation. These are critical system files.
    *   **NEVER** touch `backend/app/models/models.py` arbitrarily; it contains the core 700+ line schema.
2.  **Coding Style:**
    *   **Frontend:** Functional React components, strict TypeScript types, shadcn/ui for components.
    *   **Backend:** Type-hinted Python, Pydantic schemas for API I/O, SQLAlchemy for DB interactions.
3.  **Testing Mandate:**
    *   New features must have accompanying tests (Unit for logic, E2E for critical flows).
    *   Run `npm run typecheck` (Frontend) and `mypy` (Backend) before declaring a task complete.
4.  **Language:**
    *   Code/Comments: English (Technical standard).
    *   User Interaction: **Spanish** (User preference).
    *   Domain Data: Japanese (Names, addresses, resume fields).

## 5. Directory Structure Highlights

*   `BASEDATEJP/`: Contains raw Excel/CSV data for Japanese companies/employees (Source of Truth for imports).
*   `backend/app/api/`: Contains the 24+ API routers (e.g., `candidates.py`, `employees.py`, `payroll.py`).
*   `frontend/app/(dashboard)/`: Protected Next.js routes requiring authentication.
*   `docs/`: Extensive documentation (Architecture, Troubleshooting, Setup).
*   `.claude/`: Agent orchestration memory and prompt templates.

## 6. Critical Business Logic

*   **Candidate <-> Employee Linking:** Matching is done primarily via **Romanized Name + Date of Birth**. *Furigana* is unreliable and should not be used as a primary key for linking.
*   **Payroll:** Includes specific Japanese tax calculations, social insurance, and 3-shift timecard logic (Morning/Afternoon/Night).
*   **OCR:** The system prefers Azure CV but falls back to EasyOCR/Tesseract. OCR data is stored in JSON fields.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jokken79)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jokken79)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
