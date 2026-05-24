## vcsm

> This workspace contains three completely separate products. They share engines and a contract, but they must never be mixed.

# VCSM Workspace — Global Rules

This workspace contains three completely separate products. They share engines and a contract, but they must never be mixed.

## The Three Apps

| App | Path | Product |
|-----|------|---------|
| **VCSM** | `apps/VCSM/` | Social marketplace hybrid platform (Instagram + Airbnb) |
| **Wentrex** | `apps/wentrex/` | Standalone multi-tenant LMS SaaS |
| **Traffic** | `apps/Traffic/` | Programmatic SEO directory engine (Next.js) |

**VCSM** is the core platform — a social commerce app where creators and service providers manage actor-based identities (personal profiles or business VPORTs), post content, chat, book services, and run storefronts. External business sites like Tripoint Lock & Keys (tripointlockandkeys.com) consume their VPORT data from VCSM via Edge Function APIs, keeping their own domain and UI while VCSM remains the source of truth for business identity, services, reviews, and booking.

**Traffic** is a standalone Next.js 14 static site that generates indexable city/service/neighborhood/provider directory pages for organic search discovery, routing visitors back to the VCSM platform via deep links with tracking parameters. It currently runs entirely on mock data with no database, authentication, or engine imports — it is self-contained and deployment-ready at `traffic.vibezcitizens.com` once data is wired.

## Non-Negotiable Rules

- **Never import from one app into the other.** `apps/VCSM` and `apps/wentrex` are fully isolated products.
- **Never assume a pattern from one app applies to the other.** They share contracts and engines, but have different domain models, UI structures, and product logic.
- **Always confirm which app you are working in before making changes.** If the task is ambiguous, ask.
- **Never move features between apps.** If both apps need something similar, it belongs in `engines/` or `shared/`, not copied between apps.
- **Both apps have LMS features — they are not the same LMS.** VCSM has an embedded `/learning` route. Wentrex IS a standalone LMS SaaS. Do not conflate them.

## VCSM Architecture Contract — Mandatory Pre-Work Gate

> **Before working on anything inside `apps/VCSM/`, read this file in full:**
> `/Users/vcsm/Desktop/VCSM/zNOTFORPRODUCTION/_CANONICAL/zcontract/ARCHITECTURE.md`
>
> This contract is locked. It overrides any local assumptions, prior patterns, or inferred conventions.

### Identity — Actor-Based Only

- VCSM is actor-based. The canonical identity fields are `actorId` and `kind` (`'user'` | `'vport'`).
- **Never** scope behavior by `profileId`, `vportId`, or raw `userId`.
- **Never** expose `profileId` or `vportId` through `useIdentity()` or any public hook/controller surface.
- "Owner" always means Actor Owner — verified through `actor_owners`. There is no other ownership model.

### Screen Role Boundaries

Every file in `apps/VCSM/src/features/` must belong to exactly one layer and respect its role:

| Layer | Role | What it must NOT do |
|---|---|---|
| **Final Screen** | Route entry + identity gate only | No hooks, no computation, no data fetching |
| **View Screen** | Hooks + component composition | No business logic, no DB access |
| **Components** | Presentational only | No hooks, no data fetching, no side effects |
| **Hooks** | Lifecycle / timing / state wiring | No business rules, no direct DB access |
| **Controllers** | Business rules, ownership, permissions | No React, no UI concerns |
| **Models** | Domain shape translation, pure transforms | No side effects, no DB access |
| **DAL** | Raw Supabase access only | No business logic, no UI concerns |

### Mandatory Build Order

Always build in this order. Do not skip layers or work backwards.

```
DAL → Model → Controller → Hook → Components → View Screen → Final Screen
```

### Additional Hard Rules

- **Imports:** All new cross-folder imports must use `@/...` path aliases — never relative `../../` chains.
- **DAL selects:** Always use explicit column lists. `select('*')` is banned.
- **File length:** Keep files under 300 lines. If a file exceeds this, split it before continuing.
- **Cross-feature access:** One feature must never import directly from another feature's internals. All cross-feature access must go through adapters only.

---

## Shared Infrastructure (Safe to Consume from Both Apps)

- `engines/` — reusable domain engines (chat, identity, hydration, portfolio, reviews, booking, notifications)
- `shared/` — domain-neutral primitives (UI, utils, types)
- `contract/` — locked architecture contracts

## Dependency Direction

```
apps/VCSM     ──┐
                ├──→ engines/ ──→ shared/
apps/wentrex  ──┘
```

Apps never depend on each other. Ever.

---

## Technology Stack (VCSM)

- **Language:** JavaScript (ES Modules) — **TypeScript is BANNED**
- **UI:** React 19 + Vite
- **Styling:** UnoCSS + CSS custom properties (`--vc-*` tokens in `citizens-theme.css`)
- **State:** Zustand
- **Routing:** React Router DOM
- **Database:** Supabase (PostgreSQL + Auth + Realtime)
- **Config:** `jsconfig.json` — never `tsconfig.json`
- **Full stack rules:** `logan/platform/vcsm.platform.stack-rules.md`

### Forbidden in VCSM
- `.ts` / `.tsx` files — zero allowed
- `tsconfig.json` — project uses `jsconfig.json`
- CSS-in-JS (styled-components, emotion)
- Sass/SCSS/Less
- Redux / MobX
- Next.js / Remix
- GraphQL

---

## Architecture Layer Order

```
DAL → Model → Controller → Hook → Screen
```

- **DAL:** Raw Supabase queries, single responsibility, explicit column selection (never `.select('*')`)
- **Model:** Domain models and pure transforms
- **Controller:** Business logic orchestration, calls DALs
- **Hook:** React lifecycle management, calls controllers
- **Screen:** Pure composition, no computation, no direct DB access

---

## Theme System

Single source of truth: `apps/VCSM/src/styles/citizens-theme.css`

- All colors via `--vc-*` CSS custom properties
- Feature CSS files alias tokens into local `--feature-*` vars
- `authTheme.js` provides inline style references using `var(--vc-*)`
- Do NOT hardcode blue/slate/indigo/neutral Tailwind classes — use `white/*` opacity or `purple-*`
- Full reference: `logan/vcsm/theme/vcsm.theme.design-tokens.md`

---

## Avatar Rule

**Avatars must be SQUARE with ROUNDED CORNERS. Never circular.**

| Size | Radius | Class |
|---|---|---|
| Large (64px+) | `rounded-2xl` (16px) | Profile headers |
| Medium/Small (24-56px) | `rounded-lg` (8px) | Cards, lists, pills |
| Tiny (<24px) | `rounded-md` (6px) | Compact |

- `rounded-full` is **BANNED** for avatars
- Always use `object-cover` + `border border-white/12` + `onError` fallback to `/avatar.jpg`
- Shared component: `shared/components/ActorLink.jsx`
- Full reference: `logan/platform/vcsm.platform.avatar-rules.md`

---

## Cache System

- Shared utility: `shared/lib/ttlCache.js` — `createTTLCache(ttlMs)`
- Shared actor store: `engines/hydration/src/store.js` (Zustand, 5min TTL)
- All caches export `invalidate*()` functions for write-path cache busting
- Owner/edit modes bypass cache where applicable
- Full reference: `logan/platform/vcsm.platform.cache-recommendations.md`

---

## iOS Stacking Context Rule

**Never render `position: fixed` modals inside a parent that has:**
- `backdrop-filter` / `-webkit-backdrop-filter`
- `transform` (including `translateZ(0)`)
- `filter`
- `overflow: hidden` with `border-radius`

Always render modals as **fragment siblings**, not children of styled card containers.

---

## Identity System

- Actor-based: `actorId` + `kind` (`'user'` | `'vport'`)
- Never expose `profileId` or `vportId` through `useIdentity()`
- All domain entities scoped to `vc.actors`
- Ownership via `actor_owners`

---

## Command System

| Command | Purpose | File |
|---|---|---|
| `/Wolverine` | Main planning, routing, and execution orchestrator | `.claude/commands/Wolverine.md` |
| `/Logan` | Documentation review, drift detection, sync | `.claude/commands/Logan.md` |
| `/BUGSBUNNY` | Root cause debug command | `.claude/commands/BUGSBUNNY.md` |
| `/CAPTAIN` | Next-session order capture (ideas only) | `.claude/commands/CAPTAIN.md` |
| `/DB` | Database reviewer and analyst | `.claude/commands/DB.md` |
| `/ARCHITECT` | Repository architecture mapping & DB read audit | `.claude/commands/ARCHITECT.md` |
| `/Loki` | Runtime observability and request trace | `.claude/commands/Loki.md` |
| `/Kraven` | Performance hunter and bottleneck analysis | `.claude/commands/Kraven.md` |
| `/Venom` | Security sheriff and trust boundary review | `.claude/commands/Venom.md` |
| `/Carnage` | Database migration architect | `.claude/commands/Carnage.md` |
| `/Ironman` | Feature ownership and system responsibility | `.claude/commands/Ironman.md` |
| `/Thor` | Release commander | `.claude/commands/Thor.md` |
| `/review-contract` | Architecture contract compliance check | `.claude/commands/review-contract.md` |
| `/session-summary` | End-of-session audit log | `.claude/commands/session-summary.md` |

---

## Documentation

- All system documentation lives in `/logan/` — nowhere else
- Logan files follow `domain.system.topic.md` naming convention
- Full index: `logan/README.md`

---

## Production Safety — File Rules

### Files That Must NEVER Ship to Production

| Directory | Purpose |
|---|---|
| `.claude/` | Claude Code tooling config |
| `planning/` | Session planning files |
| `logan/` | System documentation |
| `session-summaries/` | Conversation logs |
| `db_snapshot/` | Schema snapshots and seeds |
| `debuggers/` | Dev-only debug panels |
| `apps/WT/` | Internal transfer and staging workspace — never deploy |

### `apps/WT/` — Transfer Workspace Rules

`apps/WT/` is an internal staging and transfer workspace. It is **not a product app** and must never be deployed, bundled, or referenced from any production build.

- Never import from `apps/WT/` into any other app or engine.
- Never add `apps/WT/` to any CI/CD pipeline or deployment config.
- Never create production environment variables (`.env.production`) inside `apps/WT/`.
- Code in `apps/WT/` may be experimental, in-progress, or intentionally incomplete.
- Always confirm which app you are actually working in — do not confuse `apps/WT/mine-transfer` with `apps/VCSM/` or `apps/Traffic/`.

### README.md Files
- README.md files must NOT exist in the codebase unless explicitly approved.
- The only approved README.md is `logan/README.md`.
- If a README.md is needed, ask first.

### No Scattered Documentation
- Do NOT create `.md` files inside `apps/`, `engines/`, `shared/`, or `src/` directories.
- All documentation belongs in `/logan/`.

### No Files With Spaces
- All filenames must have valid extensions and no spaces.

---

## Contract References

- `/Users/vcsm/Desktop/VCSM/zNOTFORPRODUCTION/_CANONICAL/zcontract/ARCHITECTURE.md` — **locked architecture contract for apps/VCSM/ — read before every session**
- `SECURITY_ENGINEERING_CONTRACT.md` — auth, database, infrastructure security
- `SENIOR_DEVELOPER_CONTRACT.md` — execution quality standards
- `ANTI_HALLUCINATION_ENGINEERING_CONTRACT.md` — claim verification rules
- `REAL_WORLD_ENGINEERING_OPS_CONTRACT.md` — operational engineering
- `STRATEGIC_REALITY_DEBRIEF_CONTRACT.md` — product analysis

## Wentrex Architecture Review

When the user says "review wentrex", "audit wentrex", or "run deep wentrex review", follow the spec in `apps/wentrex/REVIEW.md`.

## Traffic Architecture Review

When the user says "review traffic", "audit traffic", or "run deep traffic review", follow the spec in `apps/Traffic/REVIEW.md`.

---
> Source: [VibezCitizens/VCSM](https://github.com/VibezCitizens/VCSM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
