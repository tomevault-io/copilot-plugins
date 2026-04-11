## innovation-hack

> **Innovation Hacks** is a production-ready monorepo template optimized for hackathons. It provides a robust foundation for building full-stack applications with integrated authentication, database, and a clear architectural flow.

# GEMINI.md - Innovation Hacks Context

## 🚀 Project Overview
**Innovation Hacks** is a production-ready monorepo template optimized for hackathons. It provides a robust foundation for building full-stack applications with integrated authentication, database, and a clear architectural flow.

- **Frontend:** Next.js 14 (App Router), TypeScript, Tailwind CSS, Lucide React.
- **Backend:** FastAPI (Python), Pydantic for settings and validation.
- **Database:** Supabase (PostgreSQL) for managed data storage.
- **Authentication:** Firebase Authentication (Google Sign-In) for secure user management.

### 🏗️ Architecture & Security Flow
1. **Frontend Auth:** Uses Firebase Client SDK for Google Login.
2. **API Communication:** `axios` (in `apps/web/lib/api-client.ts`) uses an interceptor to automatically attach the Firebase JWT to the `Authorization: Bearer` header.
3. **Backend Verification:** FastAPI (in `apps/api/auth.py`) uses the `Firebase Admin SDK` to verify the JWT and extract the user's `uid`.
4. **Data Isolation:** All database queries in the backend (in `apps/api/main.py`) **MUST** be filtered by the user's `uid` (stored as `author_id` in Supabase) to ensure strict data isolation between users.

---

## 🛠️ Building and Running

### Prerequisites
- Node.js (v18+)
- Python (3.9+)
- Firebase & Supabase project credentials.

### Environment Setup
- **Frontend (`apps/web`):** Create `.env` from `.env.example`.
- **Backend (`apps/api`):** Create `.env` from `.env.example` and place `firebase-service-account.json` in the directory.

### Commands
| Task | Directory | Command |
| :--- | :--- | :--- |
| **Install Frontend** | `apps/web` | `npm install` |
| **Run Frontend** | `apps/web` | `npm run dev` (Starts at `localhost:3000`) |
| **Build Frontend** | `apps/web` | `npm run build` |
| **Install Backend** | `apps/api` | `pip install -r requirements.txt` |
| **Run Backend** | `apps/api` | `python main.py` (Starts at `localhost:8000`) |

---

## 📜 Development Conventions

### Branching Strategy
- `main`: Production-ready code (Protected).
- `feat/feature-name`: For new features.
- `fix/bug-name`: For bug fixes.
- `chore/task-name`: For maintenance or documentation.

### Commit Messages
Follow a simplified Conventional Commits format: `type(scope): brief description`
- `feat(web): add dashboard analytics`
- `fix(api): handle empty database results`
- `chore(docs): update setup instructions`

### 🚨 Critical Rules
1. **Secrets:** Never commit `.env` files or the `firebase-service-account.json`.
2. **Data Integrity:** Always use the `get_current_user` dependency in FastAPI routes to enforce authentication and access the user's `uid`.
3. **Consistency:** Ensure frontend `apiClient` is used for all backend requests to handle token refresh and authorization headers automatically.
4. **Mocking:** If Supabase is not yet configured, provide mock responses in the API but include TODOs for real persistence.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KashyapHegdeKota)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KashyapHegdeKota)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
