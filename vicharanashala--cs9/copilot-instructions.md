## cs9

> Operating guide for **all** AI agents (Claude Code, and any other agent tooling) working in this repository. Read this before making changes. Human contributors should follow it too.

# AGENTS.md

Operating guide for **all** AI agents (Claude Code, and any other agent tooling) working in this repository. Read this before making changes. Human contributors should follow it too.

The repo is a two-part app:

- `frontend/` — React 19 + Vite + Tailwind (has ESLint).
- `backend/` — Node/Express + Mongoose (no linter; `node --test` for tests).

See [CONTEXT.md](CONTEXT.md) for architecture and [CONTRIBUTING.md](CONTRIBUTING.md) for the PR template.

---

## Linting is mandatory

**Every commit must pass linting.** Do not commit, push, or open a PR with lint errors in the files you changed.

### Commands

```bash
# Lint the whole frontend (matches CI / the npm script)
cd frontend && npm run lint

# Lint only specific files you touched
cd frontend && npx eslint src/path/to/File.jsx

# Auto-fix what can be fixed automatically, then review the diff
cd frontend && npx eslint --fix src/path/to/File.jsx
```

The ESLint config is [frontend/eslint.config.js](frontend/eslint.config.js). The backend has no linter — keep its style consistent with the surrounding code instead.

### The rule for agents

1. After editing any file under `frontend/`, run ESLint on it **before** staging.
2. If ESLint reports errors, **fix them** — do not suppress with `eslint-disable` unless you can justify it in the PR description.
3. A commit that touches frontend code is "done" only when `npx eslint` passes on the changed files.
4. Treat lint errors as blocking, the same as a failing build or test.

Common errors seen in this codebase and how to resolve them (don't just silence them):

- `react-hooks/purity` — don't call impure functions like `Date.now()` / `Math.random()` during render. Read the value in an event handler, `useEffect`, or `useMemo` (with the right deps) instead.
- `no-useless-assignment` — drop initializer values that are overwritten in every branch before being read.
- `no-unused-vars` — remove the unused binding (e.g. an unused `useNavigate()` result).

---

## Pre-commit hook (hard enforcement, via Husky)

[Husky](https://typicode.github.io/husky/) runs a pre-commit hook that lints the **staged** frontend files on every commit and blocks the commit if ESLint fails. The hook source lives at [frontend/.husky/pre-commit](frontend/.husky/pre-commit).

**It auto-installs — no manual git config needed.** The `prepare` script in `frontend/package.json` runs Husky during `npm install`:

```bash
cd frontend && npm install   # installs deps AND activates the git hooks
```

So in a fresh checkout, simply installing frontend dependencies is enough to arm the hook. (If you ever need to re-arm it manually: `cd frontend && npm run prepare`.) The hook only lints files staged in the current commit, so it won't block commits that don't touch frontend code.

To bypass in a genuine emergency: `git commit --no-verify` — but the lint debt must be cleared in the same PR.

---

## Workflow expectations

- Branch off `main`; do not commit directly to `main`.
- Keep PRs scoped. Do **not** bundle unrelated local-environment tweaks (e.g. changing dev ports `5000`→`5001`, hardcoding personal DNS/IPs, connection fallbacks) into feature commits — these have repeatedly caused noise here.
- Run `cd frontend && npm run build` for frontend changes and `cd backend && npm test` for backend changes before opening a PR.
- End commit messages with the required co-author trailer when the work was AI-assisted.

## Known lint debt

As of this writing `npm run lint` reports a handful of pre-existing errors (onboarding tours, edit/delete helpers, an unused `navigate`). New work must not add to them, and touching one of those files means fixing its errors as part of your change.

---
> Source: [vicharanashala/cs9](https://github.com/vicharanashala/cs9) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
