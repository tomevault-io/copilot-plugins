## sargam-v1

> Quick reference to help AI coding agents be productive in this repo.

# Copilot instructions for sargam-v1

Quick reference to help AI coding agents be productive in this repo.

## Big picture

- This is a browser-first React/Vite app for an Indian Music Notebook (.imnb) format. The UI lives in `frontend/` (React + TypeScript + Vite + Tailwind + shadcn). Audio uses Tone.js and the domain language is `sargam-v1` (see `sargam_spec.md`).

## Key files & where to look

- Specification: `sargam_spec.md` — canonical grammar and semantics of `sargam-v1` (directives, voices, note tokens, ornaments, microtones).
- Parser implementation: `frontend/src/utils/sargam_parser.ts` — authoritative, small TypeScript parser and event model (Note/Rest/Hold/Bar, `directives`, `voices`). Use this when making changes to notation handling.
- Parser smoke test: `frontend/src/utils/test_parser.ts` — manual script that prints parse output; run with your preferred TS runner (e.g., `bun src/utils/test_parser.ts` or `npx ts-node src/utils/test_parser.ts`).
- Demo content: `frontend/public/raag_khamaj_demo.imnb` — example notebook to validate UI & playback changes.
- Google Drive integration: `frontend/src/lib/googleDrive.ts` and `frontend/src/App.tsx` (`VITE_GOOGLE_CLIENT_ID` env var). Auto-save debounces 2s and stores token in `sessionStorage`.
- Dev & build: `frontend/package.json` (scripts: `dev`, `build`, `lint`, `preview`), `frontend/Dockerfile` and `docker-compose.yml` for containerized dev (exposes port 5174).
- Linting: `frontend/eslint.config.js` — repo uses ESLint + TypeScript rules; run `npm run lint`.

## Developer workflows / commands

- Quick dev: cd `frontend` && `npm install` (or `bun install`), then `npm run dev` (app at http://localhost:5174).
- Build: `cd frontend && npm run build` (or `bun run build`). Preview with `npm run preview`.
- Docker dev: `docker-compose up` (frontend service uses `bun run dev`, binds local files and a named `frontend_node_modules` volume).
- Lint: `cd frontend && npm run lint`.

## Project-specific patterns & conventions

- TypeScript-first but some legacy JS files may exist. Path alias `@/*` → `./src/*` (see `frontend/tsconfig.json`).
- `sargam-v1` notation handling is intentionally permissive: unknown directives/ornaments should be preserved when serializing (see spec). Parser uses lowercased directive keys (e.g. `@default_duration`).
- Voices: declared using `#voice <name>`; if absent, a `default` voice is used.
- Duration defaults: set via `@default_duration` directive; parser uses 1.0 if not provided.
- Parser event model: rely on `Event` union (`note`, `rest`, `hold`, `bar`) when making changes to downstream playback code.
- UI theme: `sargam-theme` stored in localStorage; default is `light` (see `App.tsx` and `PreferenceModal` planning notes).

## Integration notes & TODOs you may find

- Google Drive: initialization must be called with `VITE_GOOGLE_CLIENT_ID`. See `initializeGoogleAPI()` for error cases; tests should mock the gapi global.
- No Python parser code is present in this repo (README mentions Python backend tools) — the canonical grammar is in `sargam_spec.md`. If adding a backend parser, follow the spec and keep behavior consistent with `frontend/src/utils/sargam_parser.ts`.

## When changing notation or playback

- Update `sargam_spec.md` if you introduce grammar changes.
- Update `frontend/src/utils/sargam_parser.ts` and add/modify `frontend/src/utils/test_parser.ts` examples that demonstrate the change.
- Add UI fixtures in `frontend/public/*.imnb` for manual verification.

## Tests & verification

- There is no formal test runner configured for parser logic (parser smoke test is a simple script). Prefer adding focused unit tests (e.g., vitest/jest) when making parser changes, and include `.imnb` examples for integration tests.

---

If anything here is unclear or you want more detail about build/test commands, config, or the grammar, tell me which section to expand or correct. ✅

---
> Source: [vivasvan1/sargam-v1](https://github.com/vivasvan1/sargam-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
