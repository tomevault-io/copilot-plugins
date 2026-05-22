## wandr2

> Single source of truth for working principles, architecture rules, and tooling. Read this before opening a PR.

# AGENTS.md — Engineering Guardrails

Single source of truth for working principles, architecture rules, and tooling. Read this before opening a PR.

> **Stage:** personal POC. Single locale, single user, single deployment. Anything that does not serve "make Phase 1 work end-to-end" is out of scope until it does.

---

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

For multi-step tasks, state a brief plan before starting:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
```

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: every changed line should trace directly to the user's request.

---

## 4. Architecture stance

- **Modular monolith.** One Next.js app at `apps/web`. Capabilities live under `apps/web/src/modules/<capability>/{domain,application,infra,web}`.
- **Capabilities are shared across pages.** A page (Home, Sport, Romantic, Food, Chat) is a thin composition of shared modules parameterized by a **preset**. If you find yourself copying a feature into a page-specific file, you are violating this rule.
- **Selective hexagonal.** Ports + adapters only for genuine external dependencies: DB, search, map, LLM, cache. Filters, sorting, ranking, presets, UI are not ports.
- **Modules are folders, not packages.** Promote a folder to a workspace package only when a second consumer (`apps/api`, `apps/mobile`) actually exists.

## 5. Layer DAG (enforced by `dependency-cruiser`)

```
web → application → domain
                 ↘
infra → domain     (infra implements ports declared in domain)
shared/* → domain | shared/*   (no upward edges)
```

**Forbidden edges** — CI fails if any of these appear:

- `domain → application | infra | web`
- `application → infra | web`
- `infra → web | application`
- `web → infra` (web only goes through application)
- Any cycle, anywhere.

**Cross-module edges** — also enforced:

- `chat/* → feed/*` allowed. The reverse is forbidden. *Chat consumes the feed; never the inverse.*
- `<page>/* → modules/*` allowed. `modules/* → app/*` forbidden.

## 6. Layer responsibilities

| Layer | Owns | Forbidden |
|---|---|---|
| `domain` | Entities, value objects, ports (interfaces), domain errors, pure business rules | Frameworks, HTTP, Prisma, React, env, time-of-day |
| `application` | Use cases, orchestration, error mapping, dependency wiring | Direct DB calls, HTTP routes, JSX |
| `infra` | Port adapters (Prisma, Mapbox, OpenAI, Redis, …) | Business rules, orchestration |
| `shared/contracts` | Pure types crossing the API/UI boundary (DTOs, view models) | Logic, methods, domain entities |
| `shared/presets` | Per-page configuration objects | Logic, data fetching, UI |
| `shared/ui` | Presentational React components consuming DTOs | Data fetching, business rules |
| `web` (Next.js `app/`) | Route handlers, page composition, hooks | Business rules, infrastructure details |

## 7. Naming

```
Entities       PascalCase nouns       Activity, Favorite, FeedQuery
Ports          IPascalCase            IActivityRepository, ILLMProvider
Adapters       <Tech>PascalCase       PrismaActivityRepository, MapboxAdapter
Use cases      <Verb><Noun>UseCase    GetFeedUseCase, AddFavoriteUseCase
DTOs           PascalCaseDTO          ActivityDTO, FeedQueryDTO
View models    PascalCaseVM           ActivityCardVM
Presets        SCREAMING_SNAKE_CASE   HOME_PRESET, SPORT_PRESET
Errors         PascalCaseError        ActivityNotFoundError
Modules        kebab-case folders     activities, feed, favorites
```

---
> Source: [HugoCllt/wandr2](https://github.com/HugoCllt/wandr2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
