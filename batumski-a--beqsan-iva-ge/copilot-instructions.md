## beqsan-iva-ge

> You are working on **BEQSAN**, an online platform for an aluminum & PVC door/window manufacturer based in Salibauri, Batumi.

# BEQSAN — Claude Code Operating Instructions

You are working on **BEQSAN**, an online platform for an aluminum & PVC door/window manufacturer based in Salibauri, Batumi.

- **Owner:** Roman Sharashidze (BEQSAN LTD)
- **Built by:** IVA (Lasha)
- **Hosting:** `beqsan.iva.ge` (public), `admin.beqsan.iva.ge` (admin), `api.beqsan.iva.ge` (API)
- **Source of truth for product vision:** [docs/kickoff.md](docs/kickoff.md)

## Workspace layout

This workspace uses a **flat split**, not a single monorepo root:

```
e:\BEQSAN.IVA.GE\
├── BACK/        # .NET 8 solution (BEQSAN.sln, src/, tests/, docker-compose.yml)
├── FRONT/       # Vite + React 18 + TypeScript SPA (public site, eventually also admin app)
├── docs/        # kickoff, ADRs, schema, API docs, questions
├── .claude/     # skill library + slash commands
└── CLAUDE.md    # this file
```

All .NET paths live under `BACK/`. All frontend paths live under `FRONT/`. Shared docs and Claude config live at the workspace root.

## How to use this codebase

Before any non-trivial task, read the relevant skill file in `.claude/skills/`. If unsure which skill applies, read [.claude/skills/INDEX.md](.claude/skills/INDEX.md).

- For **any UI work** → read `design-system/SKILL.md` **first**. No exceptions.
- For **any backend work** → read `dotnet-clean-arch/SKILL.md` **first**.
- For **configurator work** → read both `configurator-architecture/` and `3d-scene-design/`.
- For **AI features** (photo measurement, room render) → read `ai-integration/SKILL.md`.
- For **any user-facing copy** → read `georgian-ux/SKILL.md` and `content-voice/SKILL.md`.

When a skill applies, follow it. When two skills conflict, follow the more specific one and note the conflict in `docs/questions.md`.

## Architecture invariants (NEVER violate)

- **Backend:** Clean Architecture, .NET 8, MediatR + `Result<T>` pattern. Domain depends on nothing. Application depends on Domain only. Infrastructure can reference Domain + Application. Api sits on top.
- **Frontend:** React 18 + TypeScript **strict** mode. **NO** `any`. **NO** `@ts-ignore` (use `@ts-expect-error` with a reason if you must, and only as a last resort).
- All **public catalog reads** go via **Dapper**. All **admin writes** go via **EF Core**. Never mix in the same handler.
- Every endpoint returns `Result<T>` mapped to HTTP via a shared `ToActionResult()` extension. **No raw exceptions cross the API boundary** — they become `Result.Failure` and get mapped centrally.
- **Money** is `decimal(18,2)` via a `Money` value object. Never `double`/`float`.
- **Dimensions** in centimeters, stored as `int`. Never store as decimal/double.
- **Phone numbers** in E.164 format (`+995595XXXXXX`). Normalize at the edge (input handlers, SMS receivers).
- **Times** in UTC at rest; converted to `Asia/Tbilisi` only at the presentation edge.
- **Currency** is GEL (`₾`), formatted with Georgian space thousand separator + comma decimal: `1 234,56 ₾`.

## Code style

- **Backend:** nullable enabled, file-scoped namespaces, primary constructors where natural, `sealed` records for DTOs/Commands/Queries, `internal sealed` for handlers, public API surface only when needed.
- **Frontend:** function components only, named exports preferred over default, `const Component = () => ...` over `function Component()` for consistency with Tailwind/typed-children helpers, props typed inline if local — `type Props = { ... }` colocated.
- **Tailwind:** no inline `style` attributes, always use the `cn()` utility for conditionals, design tokens come from `tailwind.config.ts`. No magic Tailwind values — extend the theme instead.
- **Commits:** Conventional Commits (`feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `perf:`, `test:`, `build:`, `ci:`).
- **Branches:** `type/scope-short-description` (e.g. `feat/configurator-step-3`, `fix/order-tracking-redirect`).

## What to NEVER do

- Never expose admin endpoints without `[Authorize]` and explicit permission check.
- Never log PII (phone, email, full name) at Information level. Use Warning+ only, with masked values: `Log.ForContext("PhoneHash", Hash(phone))`.
- Never reproduce supplier names or supplier-cost pricing in **public** copy or **public** API responses. Internal admin views only.
- Never use **lorem ipsum** — write real Georgian copy. Empty-state copy has personality (see `content-voice/SKILL.md`).
- Never default to `Inter`, `Roboto`, `Open Sans`, or any other generic web font. Use the project font stack: `BPG Glaho Sans` / `BPG Mrgvlovani Caps` / `FiraGO` / `JetBrains Mono`.
- Never write English UI copy as a placeholder. Georgian is primary; en/ru come via i18next once the Georgian copy is approved.
- Never introduce a third-party dependency without a one-line justification in the PR description.
- Never use `@ts-ignore`, `as any`, `// eslint-disable` without a specific rule name + comment explaining why.
- Never commit secrets. Use `.env`, `dotnet user-secrets` (dev), Azure Key Vault or equivalent (prod).

## Languages

- **All user-facing copy:** Georgian (primary). Translations to `en` and `ru` come via i18next once the Georgian source is finalized.
- **All code, identifiers, comments, commit messages, ADRs, PR descriptions, internal docs:** English.
- **Currency:** GEL (`₾`), formatted Georgian style (`1 234,56 ₾`).
- **Date formatting:** Georgian long form for user-visible dates (`17 მაისი, 2026`), ISO 8601 in logs / API responses (`2026-05-17T15:42:00Z`).
- **24-hour time** only. Never AM/PM.

## Decision-making

- If a design choice is **documented** in `.claude/skills/` or `docs/kickoff.md`, follow it.
- If a domain rule is **undocumented**, write the question to `docs/questions.md` and pick the most defensible default in the meantime. Don't block on the question.
- If a third-party dependency is needed, flag it before adding (one-line justification).
- If two skills contradict each other, follow the more specific one and append the conflict to `docs/questions.md`.

## Workflow

1. **Read the task.** Re-read if it's vague.
2. **Read relevant skills** (see "How to use this codebase" above).
3. **State your plan** in one paragraph — what files you'll touch, what tests you'll add, what's stubbed.
4. **Implement in small commits.** Each commit should be reviewable in under 5 minutes.
5. **Self-review against the relevant skill checklist** before declaring done.
6. **Report** what's complete, what's stubbed, what's blocked, what's open.

## Working style with Lasha

- Lasha prefers **execution over clarifying questions**. When tempted to ask "should I do X or Y?", make the reasonable call grounded in `docs/kickoff.md` + `.claude/skills/`, state the choice in one sentence, and continue. He will redirect if the call is wrong.
- Reserve real questions for: **data loss risk, irreversible operations, contradictions between sources, anything affecting production infrastructure** (Cloud9.ge BATUMSKI, beqsan.iva.ge DNS, etc.).
- Reports should be **terse**. State what changed and what's next in 1-2 sentences. The diff is the source of truth.

## Infrastructure defaults (decided, not options)

These are settled. Do **not** ask Lasha to reconsider — if a tradeoff comes up, write a new ADR.

| Concern | Choice | ADR |
|---|---|---|
| Database | **SQLite** (file at `BACK/data/beqsan.db`). Postgres only if/when scale triggers fire. | [docs/adr/0001-sqlite-primary.md](docs/adr/0001-sqlite-primary.md) |
| Cache | `IMemoryCache` (built-in) behind `ICacheService` interface | inline in 0001 |
| File storage | Local filesystem at `BACK/data/uploads/` behind `IStorageService` interface | inline in 0001 |
| Logs | Serilog → Console + rolling File (`BACK/data/logs/`); Seq added later if BATUMSKI hosts it | inline in 0001 |
| Background jobs | Hangfire with SQLite storage | inline in 0001 |
| Reverse proxy (dev) | None — `dotnet run` on `:5000` | inline in 0001 |
| Reverse proxy (prod Phase 1) | Whatever BATUMSKI already runs (Caddy/IIS — decided at deploy time) | TBD |
| Containers (dev) | **None.** Single `dotnet run` boots the whole API. | inline in 0001 |

## Build & run

```
# Backend (from e:\BEQSAN.IVA.GE\BACK\)
dotnet restore
dotnet build
dotnet run --project src/BEQSAN.Api

# Health check
curl http://localhost:5000/api/v1/health
```

Frontend commands activate once `FRONT/` scaffold lands.

---
> Source: [Batumski-A/BEQSAN.IVA.GE](https://github.com/Batumski-A/BEQSAN.IVA.GE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
