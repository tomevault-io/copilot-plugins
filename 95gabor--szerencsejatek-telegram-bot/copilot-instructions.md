## szerencsejatek-telegram-bot

> Core project context for the Szerencsejáték Telegram bot (Deno + TypeScript)


# Project context

- **Runtime**: Deno (TypeScript). Prefer std library and explicit imports with versioned URLs or `deno.json` import map.
- **Purpose**: Telegram bot for Hungarian lottery games; users submit played numbers; bot notifies on new draws with results and per-user match info.
- **Constraints**: Respect **`docs/requirements.md`** (canonical **FR-/NFR-** spec; keep it in sync when product behaviour changes) and `docs/architecture.md`. Do not implement ticket purchase or gambling facilitation beyond result notification and user-owned number tracking.
- **Compliance**: Data source for results must align with documented strategy (API vs official feed vs deferred scraping). Do not add scraping without product sign-off in docs.

# Code style

- Use `async`/`await`; avoid floating promises.
- Prefer explicit types at module boundaries (handlers, storage, external APIs).
- Centralize configuration via environment variables; validate at startup.
- **Do not add trivial comments** — no restatements of what the code already says, no section banners,
  no noise. Prefer clear names and small functions. Reserve comments for non-obvious invariants,
  security or compliance notes, and links to specs when they add real context.

# Workflow

- **After each implementation iteration**, review the change set: run `deno task check`, scan for
  scope creep and regressions, and confirm the work matches `docs/` before moving on.
- **Pull request titles**: Use **Conventional Commit** format for PR titles (for example:
  `feat: add hourly deno cron draw check`, `fix: retry telegram background init`).
- **Docs and ADRs**: When behaviour or architecture changes, update the relevant files under `docs/`
  (and `README.md` / `AGENTS.md` if contributor expectations change). Record durable stack or design
  decisions in `docs/adr/` per `docs/adr/README.md`. For **new or materially changed ADRs** tied to
  architecture, **ask the user for approval** before finalizing; if they approve, set **Approved by**
  and **Approved at** (ISO 8601 UTC timestamp) in the ADR **Review** section—do not fabricate a
  reviewer or timestamp.
- **New dependencies**: When adding a **runtime** dependency (`deno.json` import map or new pinned
  `npm:` / `jsr:` in `src/`), **update ADRs in the same change**: amend the relevant ADR
  **Dependencies** section (name + version) or add a new ADR with alternatives if needed; refresh
  `docs/adr/README.md` dependency inventory. Do not rely on `deno.lock` alone as documentation.
- **ADR completeness (same PR as code)**: Do not mark work complete if you added **Zod/env
  validation**, a **new i18n strategy**, or any **documented cross-cutting pattern** without the
  matching **ADR** (and README index + inventory when a new package is pinned). Ship docs and code
  together so `docs/adr/` never lags the implementation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/95gabor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
