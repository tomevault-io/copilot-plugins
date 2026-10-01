## simpleadmin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**See `AGENTS.md` first** for the authoritative module map, build/lint commands, the web-page and
backend-service "必做清单" checklists, and commit conventions. This file adds the backend
architecture picture that AGENTS.md does not cover; it does not repeat those checklists.

## Repository shape
Monorepo, three independently-buildable modules — change only the module you're targeting:
- `api/`   — .NET backend, solution at `api/SimpleAdmin/SimpleAdmin.sln`.
- `web/`   — Vue 3 + Vite + TS admin console (npm).
- `uniapp/` — uni-app mobile client (pnpm).

## Common commands
Backend (run from `api/SimpleAdmin`):
- `dotnet build SimpleAdmin.sln` — build (multi-targets net6/7/8).
- `dotnet run --project SimpleAdmin.Web.Entry` — run the API host.

Web (`web/`): `npm run dev` · `npm run type:check` · `npm run lint:eslint` · `npm run build:pro`

Uniapp (`uniapp/`): `pnpm dev:h5` · `pnpm type-check` · `pnpm lint` · `pnpm build:h5`

There is no automated test project. Verify changes with the module's lint + type-check + build,
plus manual walk-through of the affected flow (login → dynamic route → permission → CRUD/API).

## Backend architecture (the part that spans many files)
- **Framework: MoYu** (a Furion fork; `MoYu.Pure`). The host in
  `SimpleAdmin.Web.Entry/Program.cs` is just `Serve.Run(...)`; there is no manual DI wiring of
  services. Controllers and services are discovered by convention:
  - Services implementing `ITransient` / `IScoped` / `ISingleton` are auto-registered — inject via
    constructor; don't new them up or register manually.
  - API endpoints are convention-based dynamic controllers under `SimpleAdmin.Web.Core/Controllers`
    (`System/` for system features, `Application/` for business features).
- **Layering** (host → web → domain → data → core):
  `Web.Entry` (startup) → `Web.Core` (controllers, filters, middleware, unified result/logging)
  → `Application` (business services) + `System` (system services, entities, seed data, UserManager)
  → `SqlSugar` (DbContext, CodeFirst) → `Cache` / `Core` (attributes, enums, extensions, utils).
- **Plugin projects** are optional, self-contained capability modules:
  `Background` (jobs), `MessageCenter` (messaging), `UploadCleanup`, `Plugin`. Keep new capabilities
  isolated in the matching project rather than bloating the core layers.
- **Data layer — SqlSugar, single-instance + CodeFirst + repository**
  (`SimpleAdmin.SqlSugar/Db/`). Entities derive from `BaseEntity`. On first run the app
  **auto-creates tables and seeds data** (`SimpleAdmin.System/SeedData/`, `Utils/CodeFirstUtils.cs`)
  — no migration step. To change schema, edit the entity; to change/seed defaults, edit SeedData.
- **AuthZ: RBAC + multi-org data-scope**, enforced by controller attributes
  (`[SuperAdmin]`, `[RolePermission]`). Interface-level data-scope permission is the project's
  headline feature — respect it when adding endpoints (see AGENTS.md backend checklist).
- **Config**: each project ships `<Name>.Development.json` / `<Name>.Production.json`
  (SqlSugar, System, Web, Core, Application), selected by `ASPNETCORE_ENVIRONMENT`. The DB provider
  and connection string live in `SimpleAdmin.SqlSugar/SqlSugar.Development.json`
  (SQLite/MySQL/SqlServer blocks; switch by commenting). These dev JSON files hold real
  credentials and are often locally modified — **do not commit local connection-string/secret edits.**

## Conventions worth remembering
- C# fields: `_camelCase` for private/internal, `ALL_UPPER` for const/static-readonly (`.editorconfig`).
- Front-end: UTF-8/LF/spaces; component & page entry files are `index.vue`.
- Commits: Conventional Commits `type(scope): summary`; `web`/`uniapp` enforce commitlint + lint-staged.

---
> Source: [DotNet-MoYu/SimpleAdmin](https://github.com/DotNet-MoYu/SimpleAdmin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
