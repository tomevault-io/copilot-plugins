## open-api

> Center the repository on the OpenAPI contract. Keep canonical YAML files in `spec/` (one per service, kebab-case). Generated SDKs belong in `clients/`, server stubs in `servers/`, and shared runtime helpers under `src/common/`. Store automation in `scripts/`, supplementary docs in `docs/`, and keep all automated checks inside `tests/` so CI can discover them.

# Repository Guidelines

## Project Structure & Module Organization
Center the repository on the OpenAPI contract. Keep canonical YAML files in `spec/` (one per service, kebab-case). Generated SDKs belong in `clients/`, server stubs in `servers/`, and shared runtime helpers under `src/common/`. Store automation in `scripts/`, supplementary docs in `docs/`, and keep all automated checks inside `tests/` so CI can discover them.

```
spec/
├── openapi.yaml
src/
├── common/
clients/
servers/
tests/
scripts/
docs/
```

## Build, Test, and Development Commands
Use npm scripts to keep workflows reproducible:
- `npm install` – install CLI tooling such as `@redocly/cli` and `openapi-generator-cli`.
- `npm run spec:lint` – lint and bundle the spec into `dist/openapi.yaml`.
- `npm run generate` – refresh SDKs and stubs in `clients/` and `servers/`.
- `npm test` – execute unit, contract, and regression suites.
- `npm run docs:serve` – serve the spec locally for quick visual review.

## Coding Style & Naming Conventions
Write OpenAPI in YAML 1.2 with two-space indentation and dash-separated component names. TypeScript follows ES2022 with two-space blocks; Python follows PEP 8 with four-space blocks. Keep filenames kebab-case for scripts and snake_case for fixtures. Run `npm run spec:lint` plus language-specific formatters before committing.

## Testing Guidelines
House contract suites (Schemathesis or Dredd) in `tests/contract/` and unit tests in `tests/unit/`. Name files after the feature (`get-users.spec.ts`, `users_test.py`) and colocate fixtures in `tests/fixtures/`. Regenerate fixtures via `npm run generate` when the spec changes. Always run `npm test` before pushing and explain any skipped coverage in the pull request.

## Commit & Pull Request Guidelines
Write commits in imperative mood using Conventional Commit prefixes (`feat:`, `fix:`, `docs:`). Keep spec edits separate from generated code unless the regeneration is part of the same change. Pull requests should link related issues, outline API differences, and attach lint or test output. Wait for CI to pass and secure at least one peer review before merging.

## Security & Configuration Tips
Keep secrets in `.env.local` (gitignored) and document sample values in `.env.example`. Record authentication rules in the spec `securitySchemes` section whenever adding endpoints. Include negative tests for unauthorized access to prevent regressions.
- `OPEN_DATA_DATABASE_URL` controls repository persistence; defaults to `sqlite:///data/open_data_insight.db`. Use `memory://` for ephemeral local runs and switch to a managed Postgres URL before production.
- `OPEN_DATA_OPENAI_API_KEY` must be set to call the AI analysis endpoint. Store it via `.env.local` or your secrets manager—never commit keys.

---
> Source: [Mediumwelldone4/open_api](https://github.com/Mediumwelldone4/open_api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
