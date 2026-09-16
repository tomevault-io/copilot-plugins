## golf-handicap-system

> This workspace is for the development of a golf handicap management system

# Project Name: Node/Express + React App

## Project context
This workspace is for the development of a golf handicap management system

## Core Workflow Rules
 - **Always present a written plan and wait for approval before beginning any multi-step task**

## Architecture
- Monorepo containing a Node/Express backend and a React frontend.
- Frontend Root: `./frontend`
- Backend Root: `./backend`

## Build & Run Commands
- Install All: `npm install && npm install --prefix frontend && npm install --prefix backend`
- Run Backend (Dev): `npm run dev --prefix backend`
- Run Frontend (Dev): `npm run dev --prefix frontend`
- Build Frontend: `npm run build --prefix frontend`

## Testing Commands
- Run Backend Tests: `npm test --prefix backend`
- Run Frontend Tests: `npm test --prefix frontend`
- Run Single Test: `npx jest path/to/file.test.js`

## Code Style & Rules
- Language: Strict TypeScript / ES6+ JavaScript.
- Imports: Use ES6 `import/export` on both frontend and backend.
- Node: Express routing must separate controllers from routes. Always use asynchronous error handling wrappers.
- React: Tailwind CSS for styles, functional components with hooks, native fetch/Axios for API requests.



## GitHub Workflow Instructions
- Always run local checks including: `npm run test`,  (`npm test`), `npm run lint`, `npm run type-check` before proposing a push or issue closure.
- Fetch GitHub issues using the terminal tool if requested, but do not cache issue contents.
- Keep commits atomic and use standard prefixes: `feat:`, `fix:`, `docs:`, `test:`.

---
> Source: [socx/golf-handicap-system](https://github.com/socx/golf-handicap-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
