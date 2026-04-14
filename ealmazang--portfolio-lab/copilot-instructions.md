## portfolio-lab

> - **Language:** Communicate in Spanish, but write ALL code, variable names, comments, and documentation in English.

# General Behavior
- **Language:** Communicate in Spanish, but write ALL code, variable names, comments, and documentation in English.
- **Context:** Always read `CONTEXT.md` before starting complex tasks to ensure alignment with the PRD.
- **Conciseness:** Be concise in explanations, but verbose in code comments where logic is complex.

# Backend (Python) Rules
- **Style:** Follow PEP 8 standards.
- **Type Hinting:** MANDATORY. Use `typing` module (List, Dict, Optional, Union) or standard types. No untyped function definitions.
- **Data Validation:** Use `Pydantic` models for all API schemas and complex data structures.
- **Performance:** 
  - PREFER vectorized operations (Pandas/NumPy) over `for` loops for simulation logic.
  - Avoid global state.
- **Documentation:** Use Google-style docstrings for classes and complex functions.
- **Error Handling:** Use custom exceptions found in `core/exceptions.py` (to be created) rather than generic `Exception`.

# Frontend (Next.js/React) Rules
- **Framework:** Next.js 14+ (App Router).
- **Language:** TypeScript. Use `interface` for object definitions. Avoid `any` type strictly.
- **Components:**
  - Use Functional Components.
  - Destructure props.
  - Keep components small and focused (Single Responsibility Principle).
- **Styling:** Use Tailwind CSS utility classes.
- **State:** Prefer Server Components by default. Use Client Components only when interactivity (`useState`, `useEffect`) is needed.

# Coding Standards & Best Practices
- **Naming:**
  - Variables/Functions: `snake_case` (Python), `camelCase` (TS).
  - Classes: `PascalCase`.
  - Constants: `SCREAMING_SNAKE_CASE`.
  - Boolean variables should start with `is`, `has`, `should` (e.g., `is_active`).
- **Comments:**
  - Explain the *WHY*, not the *WHAT*.
  - If a block of code is complex, add a summary comment above it.
- **Structure:**
  - "Early Return" pattern is preferred over deep nesting (`if/else`).
  - Keep business logic inside `backend/engine` or `backend/services`, NOT in `api/` routes.

# Security
- **Credentials:** NEVER hardcode passwords, API keys, or secrets. Always use environment variables loaded from `.env`.
- **Validation:** Ensure all secrets are present in `.env.example` (with dummy values) before using them.

# Git/File Operations
- Never remove existing code unless explicitly asked or if it's a refactor.
- When creating files, ensure the directory exists first.

# Testing Expectations
- **Unit Tests:** When generating or modifying code, add or update well-organized unit tests under `tests/` to ensure new logic works and existing behavior remains stable.
- **Test Updates:** If a function has existing tests, run them after changes and update the tests only when the intended behavior has changed. Distinguish failures caused by bugs from failures caused by updated requirements and adjust accordingly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EAlmazanG) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
