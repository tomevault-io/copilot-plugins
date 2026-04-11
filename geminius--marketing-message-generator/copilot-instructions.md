## marketing-message-generator

> Market-Message-Generator combines a Node.js API with a React SPA for authoring marketing copy. Follow these notes to contribute efficiently.

# Repository Guidelines

Market-Message-Generator combines a Node.js API with a React SPA for authoring marketing copy. Follow these notes to contribute efficiently.

## Project Structure & Module Organization
- `src/server.js` is the HTTP entry point; helper modules (`handler.js`, `generator.js`, `storage.js`, `validation.js`) isolate routing, drafting logic, persistence, and input checks.
- `src/ui/` contains the client (`main.jsx`, `App.jsx`, `components/`, `platforms.js`, `styles.css`). Treat it as production code—no test doubles or fixtures.
- `tests/unit/` targets the API with Node assert tests, while `tests/ui/` mirrors the client tree with Vitest + Testing Library; co-locate new specs with the feature they cover.
- `scripts/` centralizes workflows (`install`, `run`, `test`, `lint`, `build`). Extend these scripts instead of introducing bespoke commands.
- `dist/` holds generated output. Rebuild via `npm run build`; never edit artifacts directly.

## Build, Test, and Development Commands
- `./scripts/install` – install dependencies with locked versions.
- `./scripts/run api` – start the API on `PORT=3100`.
- `./scripts/run ui` – launch the Vite dev server for the React client.
- `./scripts/test` – execute server unit tests and the UI suite.
- `npm run dev` / `npm run build` / `npm run preview` – raw Vite lifecycle commands for manual control.
- `npm run lint` – run ESLint against the UI tree; resolve all warnings before review.

## Coding Style & Naming Conventions
Use Node 18+ syntax, 2-space indentation, single quotes, and trailing commas in multiline literals. Favor descriptive, hyphenated filenames (`campaign-wizard.test.jsx`). Keep modules focused and side-effect free when practical. Use Prettier defaults for JSX formatting and run `npm run lint` before pushing.

## Testing Guidelines
Name tests `<feature>.test.js|jsx` and mirror the source structure. Write deterministic cases, resetting mocks in `beforeEach`. Maintain ≥80% statement coverage; run `npm run test -- --coverage` after material changes. Tag integration-style scenarios with the `-m it` marker so they can be filtered.

## Commit & Pull Request Guidelines
Follow Conventional Commits (`feat:`, `fix:`, `chore:`) capped at 72 characters. PRs should explain intent, link issues, and list verification steps (tests, lint, relevant screenshots). Keep diffs focused, document any follow-up work, and never commit secrets or `.env`; update `.env.example` instead.

## Security & Configuration Tips
The default storage is in-memory; call out alternatives in your PR summary. Scrub fixtures of real customer data and avoid embedding credentials. Inject configuration via environment variables and document new keys in `.env.example`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/geminius)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/geminius)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
