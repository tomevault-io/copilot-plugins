## infinity-vc

> - Workspace: `apps/**`, `packages/**`

# Architecture & DDD Invariants

## Monorepo

- Workspace: `apps/**`, `packages/**`
- App principale: `apps/infinity` (`@infinity/app`)
- UI package: `packages/ui` (`@infinity.dev/ui`)

## Stack

- Backend: AdonisJS
- Frontend: React + Inertia
- Styling: Tailwind
- Langage: TypeScript
- DB cible: PostgreSQL

## DDD layering (obligatoire)

- Domain: pas de dépendance framework/ORM
- Application: use cases orchestrent, pas d'accès Lucid direct
- Infrastructure: implémente ports/repositories
- Presentation: controller/screen mappe HTTP/Inertia et délègue aux use cases

## DI

- Préférer injection constructeur
- `@inject()` Adonis autorisé
- Controller ne doit pas instancier la couche infrastructure métier directement

## Boundaries

- `packages/ui` ne doit pas importer depuis `apps/**`
- Le code métier reste dans `apps/infinity/app/{domain,application,infrastructure}`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tungdil213) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
