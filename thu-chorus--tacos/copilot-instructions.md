## tacos

> This file gives Codex and other agentic tools the repo-specific rules for TaCOS.

# AGENTS.md

This file gives Codex and other agentic tools the repo-specific rules for TaCOS.
It is based on `.cursorrules`; when the two conflict, follow this file for agent
workflow and `.cursorrules` for unchanged product/domain details.

## Project Overview

TaCOS is the Tsinghua Chorus Online System. It supports internal chorus
operations, primarily personnel management, sheet music management, events, and
authentication.

## Architecture

- `tacos_backend/`: Django 4 + Django REST Framework API.
  - `apps/authentication/`: JWT auth and profile flows.
  - `apps/personnel/`: members, instructors, titles, imports/exports.
  - `apps/sheet_music/`: PDF sheets, visibility, downloads, watermarking.
  - `apps/events/`: events, participation/admin visibility, event sheets.
  - `apps/common/`: shared utilities, permissions, responses, base behavior.
  - `config/settings/`: split settings for base, development, testing,
    production.
- `tacos_frontend/`: Vue 3 + Vite frontend.
  - `src/views/`: page-level views.
  - `src/components/`: reusable UI components.
  - `src/api/`: Axios API wrappers.
  - `src/store/`: Vuex state.
  - `src/router/`: route definitions and guards.
  - `src/utils/`: helpers and constants.
- `scripts/`: local development, formatting, backup, and maintenance scripts.
- `tacos_media/`: local uploaded/generated media.

## Tech Stack

- Backend: Python 3.9+, Django 4.x, DRF, JWT, bcrypt, PostgreSQL in production,
  SQLite in development/testing, Redis where configured.
- Frontend: Node 20+, Vue 3, Vue Router 4, Vuex 4, Axios, Vite, Tailwind CSS,
  SCSS, Element Plus plus local shadcn-vue-like components.
- Deployment: Docker, Nginx, Gunicorn, optional Aliyun OSS file storage.

## Required Workflow

- Read the existing implementation before editing. Prefer local patterns over
  new abstractions.
- Keep changes scoped to the request. Do not refactor unrelated code.
- Do not revert or overwrite user changes unless the user explicitly asks.
- Update docs when code behavior changes. Check `README.md`,
  `DEVELOP_DOCUMENT.md`, and app-specific READMEs as relevant.
- Add or update tests for API changes and core flows.
- Before committing code, run the formatter. The preferred command is:

```bash
./scripts/format_code.sh
```

If the formatter cannot run because dependencies need network access, report the
failure and run the narrow available format commands instead.

## Common Commands

Start backend:

```bash
./scripts/start_backend.sh
```

Start frontend:

```bash
./scripts/start_frontend.sh
```

Backend tests:

```bash
cd tacos_backend
python manage.py test --settings=config.settings.testing
```

Frontend tests:

```bash
cd tacos_frontend
npm run test:unit
npm run test:e2e
```

Frontend build:

```bash
cd tacos_frontend
npm run build
```

Frontend lint/format:

```bash
cd tacos_frontend
npm run format
npm run lint
```

Backend format checks/tools:

```bash
cd tacos_backend
isort . --profile black --skip-glob '*/migrations/*' --skip-glob '*/static/*' --skip-glob '*/media/*'
black .
```

## Coding Rules

- Python follows PEP 8 with Black line length 88 and isort Black profile.
- JavaScript/Vue follows the repo ESLint and Prettier configuration.
- Python indentation: 4 spaces. JavaScript/Vue indentation: 2 spaces.
- Use LF line endings and UTF-8.
- Python names:
  - files/functions/variables: `snake_case`
  - classes: `PascalCase`
  - constants: `UPPER_SNAKE_CASE`
- JavaScript/Vue names:
  - components: `PascalCase`
  - functions/variables: `camelCase`
  - constants: `UPPER_SNAKE_CASE`
  - new non-component filenames should follow the local directory convention.
- Keep backend files under roughly 800 lines. Split only when it improves
  maintainability.
- Public functions/classes should have useful documentation when behavior is not
  obvious.
- Repo-owned code comments, docstrings, and configuration-template comments
  should be concise Chinese prose. Keep required tool directives
  (`eslint-disable`, `type: ignore`, `noqa`) and product/API names unchanged.

## API And Data Rules

- API base path: `/api/v1`.
- Auth uses JWT bearer tokens.
- Prefer DRF serializers, permissions, filters, and ORM queries over ad hoc
  parsing or raw SQL.
- Validate all user input on both frontend and backend where applicable.
- Do not log passwords, tokens, or other sensitive data.
- Lists should be paginated unless an existing endpoint intentionally differs.
- Add indexes for common query fields when adding models/fields used in filters
  or visibility checks.
- For model changes, add migrations and keep them reviewable.

## Domain Rules

- Member IDs use student IDs.
- Supported voices: `S1`, `S2`, `A1`, `A2`, `T1`, `T2`, `B1`, `B2`, and
  `Other`.
- Member tiers are first team and second team.
- Mainland China phone numbers must be 11 digits and validated consistently.
- Sheet music uploads are PDF-only and capped at 20 MB.
- Sheet downloads require authenticated users and must preserve watermark and
  download-log behavior.
- Visibility and permissions are central product behavior. When changing access
  rules, update tests for admin, member, anonymous, and any relevant special
  roles.

## Frontend Guidance

- Preserve the existing Vue 3/Vite/Tailwind/component style.
- Use existing API wrappers and store modules before adding new patterns.
- Keep operational screens dense, clear, and task-focused.
- Use existing UI components and icon libraries where available.
- Ensure mobile and desktop layouts do not overlap text or controls.

## Git And PR Rules

- Use Conventional Commits:
  - `feat: ...`
  - `fix: ...`
  - `docs: ...`
  - `style: ...`
  - `refactor: ...`
  - `test: ...`
  - `chore: ...`
- Always format code before commit.
- Run relevant tests before opening a PR. If a test cannot run, state why in the
  PR notes.
- PRs should mention behavior changes, tests run, and docs updated.

---
> Source: [thu-chorus/tacos](https://github.com/thu-chorus/tacos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
