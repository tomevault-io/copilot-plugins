## molopilot

> SaaS de gestion d'établissements (restaurants, cafés, boutiques) ciblant le marché sénégalais et l'Afrique francophone.

# Molopilot — Contexte projet

SaaS de gestion d'établissements (restaurants, cafés, boutiques) ciblant le marché sénégalais et l'Afrique francophone.

## Documents de référence

- **Spec V1 :** [docs/superpowers/specs/2026-04-28-molopilot-saas-design.md](docs/superpowers/specs/2026-04-28-molopilot-saas-design.md)
- **Spec IHM Foundation :** [docs/superpowers/specs/2026-05-02-ihm-foundation-design.md](docs/superpowers/specs/2026-05-02-ihm-foundation-design.md)
- **Suivi des tâches :** [TASKS.md](TASKS.md) — toujours à jour
- **Plans d'implémentation :** `docs/superpowers/plans/`

## Règle de workflow critique

**TASKS.md et CLAUDE.md sont mis à jour dans le même commit que le code concerné.** Ne jamais terminer une tâche sans resync. Cette règle est non-négociable.

## Stack technique

| Couche           | Choix                                                              |
| ---------------- | ------------------------------------------------------------------ |
| Front            | Next.js 15 App Router (React Server Components), TypeScript strict |
| Back             | NestJS, TypeScript                                                 |
| ORM              | Prisma                                                             |
| Base             | Postgres (Neon ou Supabase Postgres, région EU)                    |
| Monorepo         | pnpm workspaces                                                    |
| Tests            | Jest (unit + intégration via Testcontainers), Playwright (E2E)     |
| Lint / format    | ESLint + Prettier (config partagée), Husky + lint-staged           |
| Offline (caisse) | Service Worker + IndexedDB via Dexie + Background Sync             |
| Auth             | argon2id (password + PIN), JWT (jsonwebtoken ou jose)              |
| Logs             | Pino                                                               |
| Erreurs          | Sentry (front + API)                                               |
| CI/CD            | GitHub Actions                                                     |
| Hébergement      | Vercel (front), Railway / Fly.io (API), Neon / Supabase (DB)       |

## Structure repo

```
molopilot/
  apps/
    web/        # Next.js — route groups (marketing) public + (auth)/(app) à venir
                # design system : tokens.css + components/ui/
    api/        # NestJS (back)
  packages/
    db/         # Prisma schema + client + migrations
    shared/     # types DTO partagés (zod schemas)
    ui/         # composants UI réutilisables (V2)
  docs/
    superpowers/
      specs/    # specs validés
      plans/    # plans d'implémentation par phase
  CLAUDE.md
  TASKS.md
```

## Conventions

- **Devise interne :** XOF (FCFA), montants stockés en cents entiers (jamais de float).
- **Tenant isolation :** double couche — Prisma extends (applicatif) + Postgres RLS (defense-in-depth). Toute table métier porte `tenant_id`.
- **Idempotence :** ventes offline → `client_uuid` UUIDv4 généré côté client, contrainte `UNIQUE(client_uuid, tenant_id)` côté DB.
- **Snapshots :** `unit_price_cents` snapshoté sur `SaleItem` à la création, immuable ensuite.
- **Stock :** `StockMovement` est la source de vérité, `StockLevel` est une projection.
- **i18n :** clés extraites dès V1, mais traduction FR uniquement.
- **Commits :** Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`).
- **Branches :** `main` = prod déployable. Travail en branches `feat/<phase>-<scope>`.
- **PR :** lint + typecheck + tests verts obligatoire avant merge.

## Phases d'implémentation (10 plans séquentiels)

0. Fondations (monorepo, Docker dev, Prisma initial, CI) ← **en cours**

**Track parallèle Frontend (IHM) :** IHM-1 (design system + landing `/`) ✅, IHM-2 (`/tarifs`), IHM-3 (`/contact`), IHM-4 (`/mentions-legales`). Préfixe `IHM-` pour éviter conflit numérotation backend.

1. Auth + multi-tenant + RLS
2. Catalogue (back-office)
3. Stock
4. Caisse online
5. Caisse offline + sync
6. Auth caissier PIN + JWT device
7. Rapports
8. Multi-établissement UI
9. Polish + observabilité

Chaque phase = un plan dans `docs/superpowers/plans/`, exécuté task-by-task, livrable et démontrable en isolation.

## Hors-scope V1

API Wave / OM, KDS, réservations, fidélité, commande en ligne, app native, Wolof, transferts stock inter-établissement, comptabilité avancée, RH. Voir § 13 du spec.

## Marché cible — rappels utiles

- Sénégal / Afrique francophone : coupures réseau fréquentes → offline-first non-négociable côté caisse.
- Modes paiement clients finaux : Cash, Wave, Orange Money. API V2.
- Devs locaux : large communauté Laravel/PHP, mais on a tranché Next.js/NestJS pour TS uniforme + écosystème PWA mûr.
- Tarif SaaS : forfait mensuel par établissement (~15 000 FCFA), pas de % CA.

## graphify

This project has a graphify knowledge graph at graphify-out/.

Rules:

- Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md for god nodes and community structure
- If graphify-out/wiki/index.md exists, navigate it instead of reading raw files
- For cross-module "how does X relate to Y" questions, prefer `graphify query "<question>"`, `graphify path "<A>" "<B>"`, or `graphify explain "<concept>"` over grep — these traverse the graph's EXTRACTED + INFERRED edges instead of scanning files
- After modifying code files in this session, run `graphify update .` to keep the graph current (AST-only, no API cost)

---
> Source: [mouhamadoulo/molopilot](https://github.com/mouhamadoulo/molopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
