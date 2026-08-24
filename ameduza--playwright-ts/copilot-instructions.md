## playwright-ts

> Playwright + TypeScript API test suite for a [RealWorld "Conduit"](https://github.com/gothinkster/realworld) clone — a Medium-like app built with Django REST Framework (backend) and Angular (frontend). Tests live in this repo; the app under test lives in `app/`.

# Copilot Instructions

## What This Repo Is

Playwright + TypeScript API test suite for a [RealWorld "Conduit"](https://github.com/gothinkster/realworld) clone — a Medium-like app built with Django REST Framework (backend) and Angular (frontend). Tests live in this repo; the app under test lives in `app/`.

## Commands

### Run tests
```bash
npx playwright test                          # all tests
npx playwright test tests/articles.spec.ts  # single file
npx playwright test -g "create article"     # single test by name
```

### View HTML report
```bash
npx playwright show-report tmp/playwright-report
```

### Start the backend (required before running tests)
```bash
# From app/ directory
python backend/manage.py runserver
# Or from repo root:
npm run serve
```

### Format
```bash
npx prettier --write .
```

### App setup (first time)
```bash
cd app
python3 -m venv .venv && .venv/Scripts/activate   # Windows
pip install -r backend/requirements.txt
python backend/manage.py migrate
```

## Architecture

```
tests/          # Playwright test specs (.spec.ts)
fixtures/       # Custom test fixtures (extends Playwright's base test)
api-services/   # API client classes wrapping Playwright's APIRequestContext
types/          # TypeScript request/response types
utils/          # (currently empty)
app/            # The application under test (Django + Angular, excluded from tsc)
```

### Key flow

1. Tests import `test` from `fixtures/test-fixture.ts`, **not** from `@playwright/test` directly.
2. Custom fixtures instantiate API service classes and inject them (e.g., `articleApi`).
3. API service classes (e.g., `ArticleAPI`) delegate HTTP calls to `BaseRequest`, which wraps Playwright's `APIRequestContext`.
4. `BaseRequest.makeRequest` always sets `Content-Type: application/json` and an `Authorization: Token <jwt>` header. `failOnStatusCode` defaults to `true`.
5. Tests target `http://localhost:8000` (configured in `playwright.config.ts` as `baseURL`).

## Conventions

- **Always extend `test` from `fixtures/test-fixture.ts`** when writing new tests so custom fixtures are available.
- **Add new API resource clients** as a class in `api-services/`, extending `BaseRequest` internally (i.e., composing it, not inheriting). Register them as fixtures in `fixtures/test-fixture.ts`.
- **Type all request/response shapes** in `types/` before using them in API service classes.
- JWT tokens are currently hardcoded in API service files. Keep this pattern consistent until an auth fixture is introduced.
- Prettier is configured with `singleQuote: true`. Run formatter before committing.
- The `app/` directory is excluded from `tsc` compilation — don't add TypeScript imports that cross that boundary.
- HTML reports are written to `tmp/playwright-report` (gitignored).

---
> Source: [ameduza/playwright-ts](https://github.com/ameduza/playwright-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
