## glens

> GLENS is a GenLayer Intelligent Contract debugger with a Next.js frontend in `frontend/` and Python GenLayer contracts in `contracts/`.

# Repository Instructions

GLENS is a GenLayer Intelligent Contract debugger with a Next.js frontend in `frontend/` and Python GenLayer contracts in `contracts/`.

- Preserve deployed v2 behavior unless the user explicitly asks to change it. The current production v2 source is `contracts/ai_debugger_v2.py`.
- Keep v2 and v3 contract addresses separate. Do not replace the deployed v2 Bradbury address without deployment proof.
- Before editing Next.js app files, read the relevant local docs under `frontend/node_modules/next/dist/docs/` and follow `frontend/AGENTS.md`.
- Do not commit generated folders, cache files, logs, environment files, or deployment output artifacts.
- Useful checks: `cd frontend && npm run lint`, `cd frontend && npm run build`, `cd frontend && npx tsc --noEmit`, and `python -m unittest discover contracts/tests`.
- Run `genvm-lint` for contract linting. It is acceptable to create the project-local `.venv` and install the official `genvm-linter` package there when release verification requires it. Do not install unrelated packages. On Windows, run `.venv\Scripts\genvm-lint.exe check contracts\ai_debugger_v3.py`, `.venv\Scripts\genvm-lint.exe typecheck contracts\ai_debugger_v3.py`, and `.venv\Scripts\genvm-lint.exe schema contracts\ai_debugger_v3.py` with `.venv\Scripts` first on `PATH`.

---
> Source: [ysnkhc/glens](https://github.com/ysnkhc/glens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
