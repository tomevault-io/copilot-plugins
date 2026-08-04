## fairbairn-co-shopify-v2

> This file is a **table of contents**. Do not inline detail here — put content in `docs/` and link it.

# Agent Map

This file is a **table of contents**. Do not inline detail here — put content in `docs/` and link it.

- **[`docs/`](./docs/README.md)** — agent operating layer (architecture, product, plans, tasks)
- **[`design/`](./design/)** — design manifest (feature → design sources), design language/system docs, and the sources themselves (Pencil/Figma/mockups — see `design/manifest.md`)
- **[`.agents/rules/`](./.agents/rules/)** — enforceable conventions (symlinked to `.claude/rules/` and `.cursor/rules/`)
- **[`.claude/commands/`](./.claude/commands/)** — worktree lifecycle, `/new-feature`, `/orchestrate`, `/babysit-prs`
- **[`.claude/agents/`](./.claude/agents/)** — subagents: `architect`, `feature-builder`, `pr-fixer`, `docs-keeper`
- **Lane guides** — nested [`apps/server/AGENTS.md`](./apps/server/AGENTS.md) (model lane) and [`apps/web/AGENTS.md`](./apps/web/AGENTS.md) / [`apps/react-native/AGENTS.md`](./apps/react-native/AGENTS.md) (view lane)

If guidance here conflicts with the linked docs, **the linked docs win** — fix the drift.

## Repository shape

```
apps/
├── server/         Hono + Drizzle + Postgres, OpenAPI at /openapi.json   :8080
├── web/            Vite + React 19 + TanStack Router + Tailwind v4      :3000
└── react-native/   Expo + Expo Router + NativeWind
packages/
├── app-config/     Identity + theme tokens — single source of truth
├── contracts/      Boundary schemas + types (zod) — server AND clients import these
├── typescript-config/  eslint-config/
docs/  design/  resources/  .agents/  .github/
```

## Rules index (load-bearing)

| Rule | One-liner |
|---|---|
| [`000-project-overview.md`](./.agents/rules/000-project-overview.md) | Stack map, ports, commands |
| [`001-co-development.md`](./.agents/rules/001-co-development.md) | Web + RN ship together, mirrored hooks |
| [`002-api-contract.md`](./.agents/rules/002-api-contract.md) | `packages/contracts` is the authoritative boundary — contract first, both sides import it |
| [`005-git-conventions.md`](./.agents/rules/005-git-conventions.md) | Conventional commits; `fix(review):` is reserved |
| [`006-app-config.md`](./.agents/rules/006-app-config.md) | Never hardcode identity; `pnpm sync-config` |
| [`008-docs-sync.md`](./.agents/rules/008-docs-sync.md) | Code + docs same commit (Stop hook backstop) |
| [`009-design-system.md`](./.agents/rules/009-design-system.md) | Designs resolve via design/manifest.md (pluggable sources); tokens flow through app-config |
| [`010-worktree-hygiene.md`](./.agents/rules/010-worktree-hygiene.md) | Mandatory worktree cleanup: push → PR → remove |
| [`011-agent-coordination.md`](./.agents/rules/011-agent-coordination.md) | Claiming, triage labels, branch naming |
| [`012-pr-loop.md`](./.agents/rules/012-pr-loop.md) | Per-lane review loops: mechanics and guards |
| [`015-pipeline.md`](./.agents/rules/015-pipeline.md) | The pipeline: human gates, roles/lanes adapter table, invariants |

## Everyday commands

```bash
pnpm install
docker compose up -d && pnpm db:push
pnpm dev                    # all apps (web :3000, server :8080, expo)
pnpm build | lint | check-types | test
pnpm sync-config            # propagate app-config identity/theme
```

pnpm 10 only — never npm/yarn. Node ≥ 22.

## Environment

- `apps/server/.env` — `DATABASE_URL` (default `postgres://app:app@localhost:5432/app`), `PORT`
- `apps/web/.env.local` — `VITE_API_BASE_URL`
- `apps/react-native/.env` — `EXPO_PUBLIC_API_URL`

Never commit `.env*` (only `.env.example`).

## The pipeline

Runs **per feature slice** — humans plan/design slice N+1 while agents build slice N. Per slice: human plans → `architect` proposes contracts (**Gate 1: ratify & freeze**) → `/orchestrate` runs the **Definition-of-Ready check** (missing artifacts → `needs-planning`, ask the human; proceed only on explicit waiver) and decomposes into lane-routed tasks (**Gate 2: ratify**) → lane authors build in worktrees → cross-lane reviewer + CI loop with the author fixing (guards in [`012-pr-loop.md`](./.agents/rules/012-pr-loop.md); fixer runs via `/babysit-prs` on the orchestrator machine by default, or in CI when the `AUTO_FIX` repo variable is `on`) → **Gate 3: human merges**. Full flow + communication patterns: [`docs/process/flow.md`](./docs/process/flow.md); enforceable spec: [`015-pipeline.md`](./.agents/rules/015-pipeline.md); setup/adapters: [`SETUP.md`](./SETUP.md).

## Review guidelines

<!-- Codex reads this section when reviewing PRs. Keep it current. -->

- Flag only P0/P1 issues: correctness bugs, security problems, broken contracts, data loss, missing error handling on the `{ error, details? }` envelope.
- Verify contract-first compliance: any boundary-shape change must land in `packages/contracts` (schemas + types), with server routes and consumers importing from it — no shape redeclared locally, no `as` casts to dodge a contract mismatch (rule 002). Client responses go through the shared schema-parsing `apiFetch(path, schema)` — no raw fetch + cast.
- Verify co-development: user-facing features touching `apps/web` should have a mirrored `apps/react-native` change or an explicit platform-scoping note in the PR body (rule 001).
- Verify tests: new endpoints need happy-path + error tests; bug fixes need a regression test (rule 004).
- Don't flag: formatting (Prettier owns it), naming taste, TODO(project) placeholders, generated files (`route-tree.gen.ts`, `theme.css`, `theme/colors.ts`).
- Don't log PII or secrets in review comments.
<!-- TODO(project): add product-specific review concerns (auth invariants, money paths, rate limits) once features exist. -->

---
> Source: [fairbo3000/fairbairn-co-shopify-v2](https://github.com/fairbo3000/fairbairn-co-shopify-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
