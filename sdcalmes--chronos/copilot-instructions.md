## chronos

> - `backend/` — Python FastAPI backend

# Agents — Repository Conventions

## Project Structure

- `backend/` — Python FastAPI backend
- `frontend/` — React + TypeScript frontend (Vite)
- `electron/` — Electron main process
- `scripts/` — Build and dev utility scripts
- `docs/` — Specifications, phase documents, plans

## Coding Standards

### Python (backend/)
- Python 3.11+
- Use `pathlib.Path` everywhere (no `os.path`)
- Type hints on all function signatures
- Linter/Formatter: `ruff`
- Type checker: `mypy`
- Test runner: `pytest`
- Raw `sqlite3` module (no ORM)
- `piexif` for EXIF, `Pillow` for image manipulation

### TypeScript (frontend/, electron/)
- TypeScript strict mode (`"strict": true`)
- Functional components and hooks only (no class components)
- Tailwind CSS for styling (no CSS modules, no styled-components)
- Use absolute imports where configured
- Vitest + React Testing Library for tests

### General
- One logical change per commit
- Tests are written alongside the code they test, in the same commit
- No `any` types. No `# type: ignore` without a comment explaining why.
- No bare `except:` in Python
- No swallowed promises in TypeScript

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sdcalmes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sdcalmes)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
