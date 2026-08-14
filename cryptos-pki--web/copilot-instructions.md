## web

> Guide for AI agents working in this repository. Pair with `CLAUDE.md` (the working agreement and

# AGENTS.md - web

Guide for AI agents working in this repository. Pair with `CLAUDE.md` (the working agreement and
hook-enforced rules). Keep this file current when the build, layout, or public API changes.

## What this is

Fleet Manager web frontend for CryptOS-PKI. React + TypeScript, built with Vite, served by manager/.

This is a leaf application (a static bundle). It is **UI-first with mock data**: there is no backend
wiring yet. The api repo only generates Go stubs today and Connect-Web is deferred, so the UI reads
typed fixtures from `src/lib/mock.ts` that stand in for the manager's gRPC responses. The whole
bundle is self-contained (fonts bundled as woff2, strict CSP, no runtime CDN) for air-gap use.

## Layout

- `src/main.tsx` - entry; mounts the theme + auth providers and the router.
- `src/App.tsx` - route table (Fleet `/`, Nodes `/nodes`, node detail `/nodes/:name`, Audit, 404).
- `src/components/layout/` - app shell: header, sidebar nav, wordmark, theme toggle, auth gate.
- `src/components/ui/` - shadcn/ui primitives (button, card, badge, separator).
- `src/context/` - `theme.tsx` (dark/light, persisted) and `auth.tsx` (browser-mTLS gate stub).
- `src/lib/` - `mock.ts` (typed Node model + fixtures) and `utils.ts` (the `cn` helper).
- `src/pages/` - the routed views. `src/test/` - vitest setup.

## Build, test, lint

- Build: `npm run build` (`tsc -b` then `vite build`)
- Test: `npm test` (vitest; no external service required)
- Lint: `npm run lint` (eslint + `prettier --check`); `npm run format` to fix
- License headers: `task license` (golic; `.golic.yaml` adds the .ts/.tsx rules)

## Conventions and gotchas

- See `CLAUDE.md` for the branch/commit/PR rules; they are enforced by the git hooks in
  `.claude/hooks` (run `bash .claude/hooks/install.sh` once per clone).
- <project-specific conventions, non-obvious constraints, and traps an agent should know>

---
> Source: [CryptOS-PKI/web](https://github.com/CryptOS-PKI/web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
