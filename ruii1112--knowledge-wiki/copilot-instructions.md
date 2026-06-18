## knowledge-wiki

> Top-level folders mirror the planned stack: `frontend/` for the React single-page app, `backend/` for the Spring Boot API, `migration/` for Flyway (or Liquibase) SQL scripts, and `docs/` for the canonical requirements. Keep UI code under `frontend/src/` with features grouped by domain (e.g., `src/articles`); structure backend code as `backend/src/main/java/com/example/knowledge/{controller,usecase,repository,model,dto,infrastructure,config,security}` so HTTP層・ユースケース・永続化・共通コンポーネントが分離される。Database changes live in Flyway-convention files such as `migration/sql/V1__init_tables.sql`.

# Repository Guidelines

## Project Structure & Module Organization
Top-level folders mirror the planned stack: `frontend/` for the React single-page app, `backend/` for the Spring Boot API, `migration/` for Flyway (or Liquibase) SQL scripts, and `docs/` for the canonical requirements. Keep UI code under `frontend/src/` with features grouped by domain (e.g., `src/articles`); structure backend code as `backend/src/main/java/com/example/knowledge/{controller,usecase,repository,model,dto,infrastructure,config,security}` so HTTP層・ユースケース・永続化・共通コンポーネントが分離される。Database changes live in Flyway-convention files such as `migration/sql/V1__init_tables.sql`.

## Build, Test, and Development Commands
Provision dependencies per module: `cd frontend && npm install` for the UI and `cd backend && ./gradlew clean build` for the API. Day-to-day loops rely on `npm run dev` (hot reload on `localhost:5173`), `npm run test` (Vitest/Jest + React Testing Library), `./gradlew bootRun` (Spring DevTools auto-reload), and `./gradlew test` (JUnit suite). Apply schema changes with `make migrate` (or `docker compose run --rm migration`) before booting the backend so tables exist locally and in CI. When using `make up`, migration runs automatically before the backend starts.

## Coding Style & Naming Conventions
Use 2-space indentation, functional React + TypeScript components, PascalCase component files, camelCase hooks/utilities, and kebab-case asset names; colocate CSS Modules or styled-components with the component they style. Backend code targets Java 25 LTS with 4-space indentation, lowercase package names, and PascalCase classes; keep controllers thin, put orchestration in `usecase`, data access in `repository`/`infrastructure`, shared domain objects under `model`/`dto`, and keep `config`/`security` isolated. Run `npm run lint` (ESLint + Prettier + TypeScript rules) and `./gradlew checkstyleMain spotlessApply` before committing to guarantee formatting consistency.

## Testing Guidelines
Frontend test files should live beside the component (`ArticleList.test.tsx`) and mock HTTP calls with MSW to avoid brittle snapshots. Backend tests rely on JUnit 5 plus Spring Boot test slices; name methods descriptively (`shouldReturnArticlesWhenTagMatches`) and use Testcontainers for MySQL integration coverage. Target ≥80% line coverage, and add regression tests whenever a bug fix touches logic already covered by requirements.

## Commit & Pull Request Guidelines
The repository is new, so adopt Conventional Commits immediately—`feat: add tag filter panel` or `fix(auth): refresh JWT on expiry` keeps the future changelog predictable. PRs must include: a one-paragraph summary, linked issue or doc section, screenshots or curl samples for UI/API changes, migration notes, and explicit test output (`npm run test`, `./gradlew test`). Rebase before requesting review and avoid mixing refactors with feature work.

## Security & Configuration Tips
Never commit `.env` files, database dumps, or production connection strings. Document required variables in `frontend/.env.example` and `backend/.env.example`, keep JWT secrets and MySQL passwords in local secret managers, and prefer `docker compose` for ephemeral MySQL instances so each agent works with isolated data.

---
> Source: [Ruii1112/knowledge-wiki](https://github.com/Ruii1112/knowledge-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
