## rehoboam-sensor-card

> This repository contains a HACS-ready Home Assistant dashboard card built with TypeScript, Lit, Three.js, and Vite. Production code lives in `src/`, browser demo code lives in `demo/`, automated tests live in `tests/` and `e2e/`, release scripts live in `scripts/`, screenshots and other static marketing assets live in `assets/`, and architecture notes live in `docs/adr/`. The HACS resource is `dist/rehoboam-sensor-card.js`; keep that built file committed so default-branch HACS installs can work before a GitHub release exists.

# Repository Guidelines

## Project Structure & Module Organization

This repository contains a HACS-ready Home Assistant dashboard card built with TypeScript, Lit, Three.js, and Vite. Production code lives in `src/`, browser demo code lives in `demo/`, automated tests live in `tests/` and `e2e/`, release scripts live in `scripts/`, screenshots and other static marketing assets live in `assets/`, and architecture notes live in `docs/adr/`. The HACS resource is `dist/rehoboam-sensor-card.js`; keep that built file committed so default-branch HACS installs can work before a GitHub release exists.

## Build, Test, and Development Commands

- `npm install` - install dependencies.
- `npm run dev` - start the Vite demo server on `127.0.0.1`; open `/demo/` to view the card scenarios.
- `npm run lint` - run ESLint and TypeScript checks.
- `npm test` - run the Vitest unit suite.
- `npm run test:e2e` - run Playwright end-to-end tests.
- `npm run build` - build `dist/rehoboam-sensor-card.js`.
- `npm run package:release` - create `dist/rehoboam-sensor-card.zip`.
- `npm run verify` - run linting, tests, build, bundle size check, packaging, and e2e tests.

## Coding Style & Naming Conventions

Use UTF-8 text, end files with a newline, and avoid trailing whitespace. Follow the existing TypeScript style: focused modules, explicit domain names, and ecosystem-standard `kebab-case` filenames for docs and assets. Let ESLint, TypeScript, and Vite define formatting and syntax boundaries.

## Testing Guidelines

Every behavior change should include automated coverage, with regression tests for bug fixes. Unit tests use `*.test.ts` under `tests/`; browser-level behavior belongs in `e2e/`. Keep tests deterministic and independent of network services unless they are explicitly integration tests.

README screenshots should come from the local demo scenarios, not external mockups. Hide the demo controls before capturing card screenshots so the images show only the card surface.

## Commit & Pull Request Guidelines

Use concise Conventional Commit subjects such as `feat: add sensor editor validation` or `fix: preserve unavailable sensor gaps`. Keep commits narrowly scoped. Pull requests should explain the problem and solution, list verification performed, link relevant issues, and include screenshots or recordings for visible changes.

## Security & Configuration

Never commit secrets, credentials, local environment files, generated test output, or machine-specific configuration. Keep `.env.example` sanitized if one is ever added. Generated release artifacts are ignored except for `dist/rehoboam-sensor-card.js`, which is intentionally tracked for HACS.

---
> Source: [Vovoronale/rehoboam-sensor-card](https://github.com/Vovoronale/rehoboam-sensor-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
