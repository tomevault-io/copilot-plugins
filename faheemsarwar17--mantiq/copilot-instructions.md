## mantiq

> **Mantiq** is a comprehensive learning platform for Intel-8086 Assembly programming, inspired by LeetCode and TIS-100. It features a unique blend of visual novel elements, 3D world exploration, and a specialized assembly execution engine.

# GEMINI.md - Mantiq Project Context

## Project Overview
**Mantiq** is a comprehensive learning platform for Intel-8086 Assembly programming, inspired by LeetCode and TIS-100. It features a unique blend of visual novel elements, 3D world exploration, and a specialized assembly execution engine.

### Key Technologies
- **Frontend**: React 19, TypeScript, Vite, Tailwind CSS.
  - **Graphics**: Three.js (via React Three Fiber & Drei), Framer Motion.
  - **State Management**: Zustand.
  - **Routing**: React Router 7.
- **Backend**: Python 3.x, FastAPI, SQLAlchemy (ORM), Alembic (Migrations).
  - **Authentication**: JWT (Access & Refresh tokens) with Role-Based Access Control (RBAC).
- **Core Engine**: Custom Intel-8086 Emulator and Assembly Parser implemented in TypeScript (`src/engine/Emulator8086.ts`, `src/utils/assemblyParser.ts`).
- **Database**: SQLite (default for development), designed for MySQL.

### Architecture
- **Hybrid Execution**: Assembly code is parsed and emulated on the frontend for real-time feedback (Story Mode/Sandbox). For competitive puzzle submissions, the backend securely validates code by spawning a `ts-node` process that reuses the frontend's `Emulator8086` logic against server-side test cases.
- **Modular Design**: 
  - `src/components/Chapter1`: Visual Novel and 3D exploration components.
  - `src/engine/`: Core emulator and grid management logic.
  - `backend/routers/`: Modular API endpoints for Auth, Questions, and Forum.

---

## Building and Running

### Prerequisites
- Node.js (v18+)
- Python (v3.9+)
- `npx` (for backend execution logic)

### Frontend (Root Directory)
1.  **Install dependencies**:
    ```bash
    npm install
    ```
2.  **Start development server**:
    ```bash
    npm run dev
    ```
3.  **Run tests**:
    ```bash
    npm run test
    ```
4.  **Build for production**:
    ```bash
    npm run build
    ```

### Backend (`/backend` Directory)
1.  **Set up virtual environment**:
    ```bash
    cd backend
    python -m venv venv
    source venv/bin/activate # Win: venv\Scripts\activate
    ```
2.  **Install dependencies**:
    ```bash
    pip install fastapi uvicorn sqlalchemy alembic python-jose[cryptography] passlib[bcrypt] python-multipart pydantic[email]
    ```
3.  **Environment Variables**: Create a `.env` file in `backend/` based on `auth_utils.py` defaults.
4.  **Run migrations**:
    ```bash
    alembic upgrade head
    ```
5.  **Seed initial data**:
    ```bash
    python seed_questions.py
    ```
6.  **Start API server**:
    ```bash
    uvicorn main:app --reload
    ```

---

## Development Conventions

### Coding Style
- **TypeScript**: Use strict typing. Prefer functional components and hooks. Follow the modular structure in `src/`.
- **Python**: Follow PEP 8. Use Pydantic schemas for request/response validation.

### State Management
- Use **Zustand** for global state (Auth, Game progress, Achievements).
- Use local `useState` for component-specific UI state.

### Assembly Engine
- Any changes to `Emulator8086.ts` or `assemblyParser.ts` must be verified against existing tests (`src/__tests__/`).
- The backend relies on these files being present and compatible with `ts-node`.

### Graphics & UI
- Maintain the "broken-system/technical" atmosphere using the custom UI diagnostics overlay and glitch effects.
- Visual Novel backgrounds are located in `public/vn-backgrounds/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/faheemsarwar17)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/faheemsarwar17)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
