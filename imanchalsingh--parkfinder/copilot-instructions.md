## parkfinder

> Two independent workspaces, no root scripts:

# AGENTS.md

## Project Structure

Two independent workspaces, no root scripts:
- `client/` — React 19 SPA (Vite + TypeScript + Tailwind CSS v4)
- `server/` — Express 5 API (Node, ESM modules, MongoDB/Mongoose)

Root `package.json` only holds shared vitest devDependencies (no scripts).

## Commands

### Client
```bash
cd client
npm run dev       # Vite dev server on :5173
npm run build     # tsc -b && vite build
npm run lint      # eslint .
```

### Server
```bash
cd server
npm run dev       # nodemon server.js on :5000
npm run test      # vitest run
node seed.js      # seed MongoDB with parking slots
```

### Tests
- Root and both subdirectories use vitest.
- Client test config: none (uses vitest defaults). Add `vitest.config.ts` if you need aliases or setup files.
- Server: `vitest run` from `server/`. Uses supertest + @faker-js/faker for API tests.

## Environment

Server requires a `.env` in `server/` (see `.env.example`). Critical vars validated at startup — server exits if missing:
- `JWT_SECRET`, `ADMIN_SECRET` (32+ char random strings)
- `MONGO_URI` (default: `mongodb://127.0.0.1:27017/parkfinder`)
- `SMTP_*` vars for Nodemailer

Client needs `client/.env` with `VITE_API_URL=http://localhost:5000` (optional — Vite proxy defaults to localhost:5000).

## TypeScript

`client/tsconfig.app.json` — strict mode, `noUnusedLocals`, `noUnusedParameters`, `erasableSyntaxOnly`. Fix all strict violations; do not disable strict checks.

## Gotchas

- **Express 5** — not Express 4. Error handlers use `(err, req, res, next)` but async route handlers are auto-caught (no manual `try/catch` needed with `express-async-handler`).
- **Tailwind CSS v4** — uses `@tailwindcss/vite` plugin, not PostCSS. No `tailwind.config.js` needed; config goes in CSS via `@theme`.
- **ESM everywhere** — both workspaces use `"type": "module"`. Use `import`/`export`, not `require`.
- **Vite proxy** — client dev server proxies `/api` to backend. Do not hardcode `localhost:5000` in client code; use relative `/api` paths.
- **No root lint/typecheck** — lint only runs from `client/`. Server has no lint script.
- **No CI workflows** — `.github/` directory is empty. No pre-commit hooks configured (`.husky/` exists but is empty).

## Skills

Local skills in `.agents/skills/` — invoke via `/skill-name`:

| Skill | What it does in this repo |
|-------|--------------------------|
| `conventional-commit` | Inspects `git diff`, drafts a Conventional Commits message (`feat:`, `fix:`, etc.), and commits it |
| `nodejs-express-server` | Guides adding routes, middleware, and auth to the Express 5 `server/` API |
| `react-testing-library` | Writes component tests for `client/` using React Testing Library queries and user-event |
| `typescript-advanced-types` | Applies generics, mapped types, and utility types to tighten TypeScript in `client/` |
| `wcag-audit-patterns` | Audits `client/` UI components against WCAG 2.2 and gives remediation steps |

## Conventions

- Branch naming: `feature/*`, `fix/*` (per README contributing guide)
- Commits: conventional commits (`feat:`, `fix:`, etc.)
- Icons: Lucide React
- State: React Context (AuthContext), no Redux
- HTTP client: Axios

---
> Source: [imanchalsingh/parkfinder](https://github.com/imanchalsingh/parkfinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
