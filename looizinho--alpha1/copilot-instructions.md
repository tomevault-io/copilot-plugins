## alpha1

> - `README.md` holds the project overview; no source modules are defined yet.

# Repository Guidelines

## Project Structure & Module Organization
- `README.md` holds the project overview; no source modules are defined yet.
- Add future code under a clear top-level folder (e.g., `src/` or `app/`), tests under `tests/` or `__tests__/`, and assets under `assets/`.
- Keep new directories shallow and name them for domain areas (e.g., `src/auth/`, `src/billing/`).

## Build, Test, and Development Commands
- No build or test commands are configured yet.
- When you add tooling, document the exact commands here (for example, `npm run build`, `npm test`, or `make dev`) and what each does.

## Coding Style & Naming Conventions
- Default to 2-space indentation for JSON/YAML and 4-space indentation for code unless the chosen language or formatter specifies otherwise.
- Use clear, descriptive names: `UserSession` (types/classes), `user_session` (files), `userSession` (variables), `user-session` (URLs/CLI flags).
- If you add a formatter or linter (e.g., Prettier, Black, ESLint), include it in this section and run it before committing.

## Testing Guidelines
- No testing framework is configured yet.
- When adding tests, prefer colocated naming like `foo.test.ts` or folder-based `tests/foo_test.py`, and document the exact command to run them.
- Aim for fast, deterministic tests; avoid external network dependencies unless explicitly mocked.

## Commit & Pull Request Guidelines
- Current history includes a single commit message: "Initial commit". Keep commit messages short, imperative, and scoped (e.g., "Add auth middleware").
- Pull requests should include: a concise summary, linked issue (if any), and test evidence (command + result). Include screenshots for UI changes.

## Security & Configuration Tips
- Store secrets in local env files (e.g., `.env`) and add them to `.gitignore`.
- Document required environment variables in `README.md` with example values.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/looizinho)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/looizinho)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
