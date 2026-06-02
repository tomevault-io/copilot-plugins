## hushh-tech-website

> Use this file as the GitHub-wide onboarding context for Copilot cloud agent, Copilot code review, and repository chat.

# Hushh Tech Copilot Instructions

Use this file as the GitHub-wide onboarding context for Copilot cloud agent, Copilot code review, and repository chat.

## Grounding Order

When working in this repository, ground decisions in this order:

1. `AGENTS.md`
2. `QODO.MD`
3. `.pr_agent.toml`
4. `best_practices.md`
5. `docs/project_context_map.md`
6. `README.md`, `CONTRIBUTING.md`, `SECURITY.md`
7. Checked-in code and workflows on the base branch

Treat `main` as the current source of truth for runtime behavior, CI policy, and deploy policy. If a behavior is not visible in the repository, say that the evidence is not shown here instead of inferring hidden services or hidden configuration.

## Repository Shape

- This is a Hushh Tech web repository built as a `Vite + React` SPA under `src/`.
- The production web runtime is `server.js` on Cloud Run.
- API handlers live under `api/` and are loaded by `server.js`.
- Shared backend helpers live under `api/shared/`.
- Supabase migrations, functions, and local assets live under `supabase/`.
- Additional service deployments live under `cloud-run/`.
- CI and deploy policy live under `.github/workflows/`, `scripts/ci/`, and `scripts/security/`.

Do not use `.cursorrules` as the stack or runtime source of truth for this repository.

## High-Signal Code Map

- Frontend entrypoint: `src/main.tsx`
- Frontend router root: `src/App.tsx`
- Frontend routes and UI: `src/pages/**`, `src/components/**`
- Auth and token lifecycle: `src/auth/**`, `src/services/authentication/**`
- Browser services: `src/services/**`
- Web runtime and SPA fallback: `server.js`
- Backend request handlers: `api/**`
- Backend shared utilities: `api/shared/**`
- Data-contract and migration surfaces: `supabase/**`
- Deploy and validation governance: `.github/workflows/**`, `scripts/ci/**`, `scripts/security/**`

## Non-Negotiable Hushh Rules

- `VITE_*` variables are browser-visible. Do not treat them as secrets.
- Route changes under `api/**` are incomplete if `server.js` runtime wiring is not kept in sync.
- Auth, env, workflow, deploy, and migration changes require proof, not just intent.
- `PR Validation` is the authoritative code-executing merge gate. Advisory reviewers do not replace it.
- Prefer narrow, evidence-backed findings. Do not invent architecture, hidden services, or vendor behavior that is not checked in.

## Validation Commands

Use the smallest relevant set of commands first:

```bash
npm test
npx tsc --noEmit
npm run build:web
npm run env:check
npm run lint:ci
npm run security:gitleaks
npm run security:audit
npm run smoke:ci
```

For local runtime work:

```bash
npm run dev
npm run dev:api
```

## Working Style

- Start with the grounding files above before broad repo search.
- Search only when those files are incomplete, contradicted by current code, or the task is path-specific.
- Keep suggestions production-relevant: correctness, security, auth, env wiring, CI/deploy safety, data contracts, and missing proof.
- If a task touches `server.js`, `api/**`, `src/auth/**`, `src/services/authentication/**`, `supabase/**`, `.github/workflows/**`, or `scripts/ci/**`, treat it as high-risk and validate more aggressively.

---
> Source: [hushh-labs/hushh_Tech_website](https://github.com/hushh-labs/hushh_Tech_website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
