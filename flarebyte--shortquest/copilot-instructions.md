## shortquest

> - `index.js`: Core library (rule-driven HTTP request builder).

# Repository Guidelines

## Project Structure & Module Organization
- `index.js`: Core library (rule-driven HTTP request builder).
- `test/`: Mocha tests, fixtures (`dummy*.txt`) and integration-style assertions.
- `shortquest.schema.json`: JSON Schema for rules; see `RULES.md` for triggers/actions.
- Tooling: `Gruntfile.js` (tasks), `.jshintrc` (lint rules), `.travis.yml` (legacy CI), `package.json`.

## Build, Test, and Development Commands
```bash
npm install           # install dependencies
npm test              # run lint (jshint) and tests (mocha via grunt)
grunt jshint         # lint only
grunt mochacli       # run mocha tests with nyan reporter
grunt watch          # watch files and re-run lint/tests
```

## Coding Style & Naming Conventions
- JavaScript (Node, ES5, `'use strict'`). Match existing spacing (currently 4 spaces).
- Keep functions small and pure; validate inputs with `Joi`.
- Naming: camelCase for variables/functions (`setAuthorization`), UPPER_SNAKE for constants.
- Lint: adhere to `.jshintrc` (curly braces, `eqeqeq`, no unused/undef). Run `grunt jshint` before pushing.

## Testing Guidelines
- Frameworks: Mocha + Chai (`assert`), HTTP mocking with `nock`.
- Location: `test/*.js`. Prefer descriptive `describe`/`it` names (e.g., `it('must set authorization bearer', ...)`).
- Add fixtures under `test/` as needed; avoid real secrets.
- Run locally with `npm test`. Add tests for new triggers/actions and schema changes.

## Commit & Pull Request Guidelines
- Commits: short, imperative, and scoped (e.g., `Add json-schema model`, `Fix OAuth RSA handling`).
- Reference issues in the body (e.g., `Fixes #123`).
- PRs: include a clear summary, motivation, before/after behavior, and test coverage. Add screenshots only if relevant to docs.
- Versioning: semantic. Maintainers use `grunt bump` for releases; do not bump versions in feature PRs.

## Security & Configuration Tips
- Never commit real credentials, tokens, or certificates. Use dummy fixtures and file paths in tests.
- When adding SSL features, read files from paths (see `agentOptions.*Path`) and keep content out of source control.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/flarebyte)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/flarebyte)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
