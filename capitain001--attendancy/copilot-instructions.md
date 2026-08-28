## attendancy

> attendancy/                    ← racine monorepo (bun workspaces + TurboRepo)

# Conventions du projet

## Structure monorepo

```
attendancy/                    ← racine monorepo (bun workspaces + TurboRepo)
├── apps/
│   ├── web/                   ← app Next.js 16 (tout le backend + frontend web)
│   └── desktop/               ← app Vite + Tauri v2 (shell desktop/mobile)
├── packages/
│   ├── types/                 ← DTOs partagés (@attendancy/types)
│   └── planning/              ← hooks React Query + vues planning (@attendancy/planning)
├── docs/
│   └── techs/                 ← stack.md, migration-monorepo.md
└── turbo.json
```

Tout le code Next.js (src/, prisma/, scripts/, summary/) est sous `apps/web/`.
Les chemins ci-dessous sont **relatifs à `apps/web/`**.

## Avant d'écrire du code

- `apps/web/src/services/**` → lire **obligatoirement** `docs/skills/service-module-pattern/SKILL.md`
  + `apps/web/src/services/SERVICE_CONTEXT.md` + le `CLAUDE.md` du service concerné avant toute action.
  Consulter `apps/web/summary/<service>.json` pour un aperçu rapide des fonctions existantes
  sans ouvrir chaque fichier.
- `apps/web/src/app/**` (pages RSC, layouts) → lire `docs/skills/nextjs-ppr/SKILL.md` (règle
  `connection()` / Suspense — contrainte PPR `cacheComponents: true`).
- `apps/web/src/hooks/data/**` → lire `apps/web/src/hooks/CLAUDE.md` + `src/hooks/entity/usage.md`.
- `apps/web/src/store/**` → lire `apps/web/src/store/STORE_CONTEXT.md` (quand un store vs React Query vs RSC).
- `docs/**` → lire `docs/DOCS_CONTEXT.md` (quel document va dans quel dossier).
- `apps/web/prisma/schemas/**` → lire `apps/web/prisma/schemas/README.md` (découpage multi-schema).
- `apps/web/prisma/post-migrate/**` → lire `apps/web/prisma/post-migrate/README.md` (SQL hors Prisma).
- Scripts disponibles → `docs/cmd/generators.md` (api, summary, types, naming — commandes complètes).
  Lancer depuis `apps/web/` : `cd apps/web && bun run generate:api:svc -- <service>`
- `packages/types/**` → DTOs cross-platform — pas de dépendance Prisma, types simples uniquement.
- `packages/planning/**` → hooks React Query + vues — appels HTTP vers `/api/*`, pas de server actions.
- Tout nouveau service créé → écrire son `CLAUDE.md` avant le premier commit.

## Stack (non négociable)

### apps/web
Next.js 16 (`cacheComponents: true` — PPR actif) · React 19 · Prisma v7
multi-schema (adapter pg) · Supabase (auth) · Tailwind v4 + shadcn/ui + base-ui ·
TypeScript strict · Valibot · Vitest .

### apps/desktop
Vite 6 · React 19 · Tauri v2 · @attendancy/planning · @attendancy/types.

### packages
TypeScript strict · pas de framework UI · pas de Prisma · pas de Valibot.

## Invariants (toujours enforced)

- `orgId` extrait du token auth serveur UNIQUEMENT — jamais du body/query/headers.
- Prisma uniquement dans `apps/web/src/services/*/database/` — jamais ailleurs.
- `"use server"` uniquement dans `apps/web/src/services/*/actions/` — jamais sur les utilitaires.
- Frontend (pages RSC incluses) → toujours via `actions/`, jamais `database/`.
- Actions : préfixe `get*` (jamais `list*`), retour `{ data }` / `{ error: string }`.
- Pages RSC sans `getUserInfo()` direct → `await connection()` (next/server) en
  tête (contrainte PPR de cacheComponents).
- Chaque service maintient son `CLAUDE.md` à jour.
- Composants clients : jamais d'appel direct à une server action — toujours via
  un hook `apps/web/src/hooks/data/<domain>/` (useCrudEntity/useEntity).
- `packages/planning` et `packages/types` : jamais de server actions, jamais de Prisma.
  Appels HTTP uniquement (`apiFetch` vers `/api/*`).

## Naming (non négociable)

### Suppressions
- **Soft delete** (`deletedAt`) → préfixe `remove` · événement `*_REMOVED`
  - ✅ `removeRoom`, `GROUP_REMOVED`
  - ❌ `deleteRoom`, `GROUP_DELETED`
- **Hard delete** (row supprimé physiquement) → préfixe `delete` · événement `*_DELETED`
  - ✅ `deleteRoom`, `GROUP_DELETED`

### Lecture
- Toujours `get*` — le préfixe `list*` est interdit partout (DB, actions, hooks).
  - ✅ `getRooms`, `getSchedulesAction`
  - ❌ `listRooms`, `listSchedulesAction`

### Typage Prisma
- `Awaited<ReturnType<typeof fn>>` — jamais `Prisma.PromiseReturnType<>`.

### ActionResponse
- Retour discriminé : `{ data: T } | { error: string }` — jamais `{ data?: T; error?: string }`.
- Narrowing : `if ('error' in result)` — jamais `if (result.error)`.

### Vérification post-travail

Après chaque session sur `apps/web/src/services/**` — lancer depuis `apps/web/` avant le commit :

```bash
cd apps/web
bun run check:naming:svc -- <service>
bun run check:types:svc -- <service>
bun run generate:api:svc -- <service>
bun run api:check
```

- `check:naming` : conventions soft/hard delete, préfixe `get*` (exit 0, non bloquant).
- `check:types` : vérifie que les `Awaited<ReturnType<...>>` sont dans `types.ts` (exit 0).
- `generate:api` : met à jour l'index `.api/` du service — obligatoire après toute
  mutation d'action (ajout, renommage, suppression).
- `api:check` : valide la cohérence cross-service — obligatoire avant commit.

Optionnel (lecture IA à jour) :
```bash
bun run generate:summary:svc -- <service>
```

## Propriété des modèles

Un modèle Prisma appartient à **un seul service**. Prisma ne s'utilise que dans
le `database/` de ce service. Un service consommateur appelle les fonctions de
l'owner — il ne refait pas un `prisma.x.findMany()` sur un modèle étranger.

## Contexte projet

- `docs/techs/stack.md` — stack actuelle et cible de migration (Bun, Tauri, Expo)
- `docs/techs/migration-monorepo.md` — plan de migration monorepo, phases et décisions

<!-- ⚠ À ÉTENDRE PAR PROJET — référencer ici les fichiers docs/context/ du
     domaine métier, à lire avant d'écrire de la logique métier. Exemple :
- `docs/context/01_overview.yaml` — vision produit
- `docs/context/06_business_rules.yaml` — règles métier RULE-*
-->

---
> Source: [Capitain001/attendancy](https://github.com/Capitain001/attendancy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
