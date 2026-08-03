## ai-coding-platform

> > Project context for Claude Code. This file is the **scope of record**. Nested

# Relay

> Project context for Claude Code. This file is the **scope of record**. Nested
> `CLAUDE.md` files (`apps/backend`, `apps/frontend`, `packages/shared`) hold
> only what is specific to their area — don't repeat this file there.

## What Relay is

Relay is a **control plane that orchestrates a pipeline of AI agents** on top of
**Linear** (tickets) and **GitHub** (code). The user thinks and validates; the
agents do the work; Relay is the cockpit to drive it — including from a phone,
away from the dev machine.

The flow, end to end:

```
Linear ticket ─▶ AI prep ("AI ready") ─▶ Implementation agent ─▶ Code review agent
   ─▶ Security review agent ─▶ GitHub branch + Vercel preview ─▶ Human review ─▶ Merge
```

**Linear and GitHub stay the source of truth.** Relay reads/writes Linear via its
API, drives GitHub via a GitHub App, and surfaces Vercel previews. Relay never
becomes the canonical store for tickets or code.

### Pipeline stages (8, locked)

`todo` → `ai-ready` → `coding` → `review` → `security` → `preview` → `validated` → `merged`

These stage names are canonical across the whole system: data model, API,
frontend badges, and the design tokens (`docs/design/project/design-tokens.css`).
They live as an enum in `@relay/shared`.

## Audience — multi-user SaaS

Relay is **multi-tenant from day one**.

- The tenant unit is the **Workspace**. A Workspace has **one owner + N members**.
- **Every entity carries `workspaceId`.** No exceptions. This is the tenancy
  boundary and the default query filter.
- Users authenticate to Relay, then connect provider accounts via **OAuth**
  (Linear, GitHub, Vercel) and bring their **own API keys** (BYOK — Anthropic /
  OpenAI).
- **Connections and secrets are scoped to the Workspace**, configured by the
  owner, usable by members.
- **All secrets are encrypted at rest.** Never store provider tokens or API keys
  in plaintext; never log them.

## Surface — responsive, desktop + mobile both first-class

- The frontend is a **Next.js web app**, deployed on Vercel.
- The visual language is a **dense desktop "cockpit"** (top nav, terminal-like
  logs, generous data density) — see the design.
- **Mobile is first-class, not an afterthought.** Key screens get dedicated
  mobile layouts so Relay is usable one-handed when away from the desk.
- The design ships **3 themes** (light canonical, dim, dark) and **2 densities**
  (cozy default, compact). Light is the canonical baseline.

## Design

A complete hi-fi design exists and is **vendored in `docs/design/`**:

- `docs/design/project/design-tokens.css` — the locked design system (color,
  type, spacing, radii, shadows, themes, densities). This is the source of truth
  for the frontend's CSS variables.
- `docs/design/project/style-guide.html` — the component gallery.
- `docs/design/project/*.html` — 15 screens (login, register, onboarding,
  dashboard, tickets, ticket-detail, pipeline, agents, review, request-changes,
  failures, project, settings).
- `docs/design/project/DESIGN_BRIEF.md` — original design brief.

**Important — the brief predates the current product direction.** The brief
assumes single-user / desktop-only. The product is now **multi-user SaaS** and
**responsive (mobile first-class)** — see above. Treat the design as the **visual
language and component system**, and adapt the product frame: add what the brief
omits (workspace switcher, member management, multi-tenant auth, mobile layouts).
The tokens, components, and 8 stages carry over unchanged.

The mockups are HTML/CSS prototypes. **Recreate them in React/Tailwind; don't copy
the prototype's internal markup.** Match the visual output.

## Open decisions (not yet settled — do not assume)

- **Where implementation agents execute.** Candidates: GitHub Actions (CI),
  a worker on Relay's own infra, or a hosted agent service. Not decided. Don't
  bake an assumption into the data model or API beyond what's reversible.

## Out of scope for now

- Billing / subscriptions (Stripe). Note: **cost budgets** — monthly cap,
  per-ticket cap, iteration cap — ARE in scope (Settings); that's cost control,
  not billing.
- Native mobile apps (the responsive web app covers mobile).
- A no-code/marketing surface.

## Architecture

Monorepo via **Turborepo + npm workspaces**:

| Workspace | Package | Role |
|---|---|---|
| `apps/backend` | `backend` | NestJS 11 — the orchestrator (API, pipeline, integrations). |
| `apps/frontend` | `frontend` | Next.js 16 / React 19 / Tailwind 4 — the cockpit UI. |
| `packages/shared` | `@relay/shared` | Shared enums, model interfaces, Zod DTOs. No runtime deps beyond zod. |

- **Datastores:** MongoDB + Redis, local via Docker Compose.
- **Local-dev port remap (intentional):** Mongo → host **27018**, Redis → host
  **6380** (not the defaults 27017/6379), because this machine runs other project
  containers on the default ports. `.env` / `.env.example` point at 27018 / 6380.

## Conventions (global)

- **TypeScript strict everywhere.** No `any` unless truly unavoidable; prefer
  precise types, generics, or `unknown`. `strict` + `noImplicitAny` are on in
  `tsconfig.base.json`.
- **Multi-tenancy is a hard rule:** every new collection/entity carries
  `workspaceId`; every query is scoped by it.
- **Secrets:** encrypt at rest, never log, never return to the client.
- **Shared types live in `@relay/shared`** — don't redefine model shapes or enums
  in an app when they're cross-cutting.
- **Charts:** use **Chart.js** for any data visualization.
- Node ≥ 20. Package manager: **npm** (`npm@11.4.1`).

## Commands

```bash
npm run dev         # turbo run dev — all apps
npm run build       # turbo run build
npm run lint        # turbo run lint
npm run typecheck   # turbo run typecheck
npm run test        # turbo run test
npm run format      # prettier across the repo

npm run db:up       # docker compose up -d  (Mongo 27018 + Redis 6380)
npm run db:down     # docker compose down
```

## Where to look

- **Backend specifics** → `apps/backend/CLAUDE.md`
- **Frontend specifics + design system** → `apps/frontend/CLAUDE.md`
- **Shared types** → `packages/shared/CLAUDE.md`
- **Design** → `docs/design/`

---
> Source: [mathysth/ai-coding-platform](https://github.com/mathysth/ai-coding-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
