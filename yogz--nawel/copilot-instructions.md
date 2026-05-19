## nawel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Companion docs (read on demand) :
>
> - **`PROJECT_STANDARDS.md`** — règles détaillées (Server Actions, A11y, perf, i18n)
> - **`DESIGN_SYSTEM.md`** — tokens, motion vocabulary, palette CoList + Sortie
> - **`ANALYTICS_AUDIT.md`** — état tracking produit, cadre AAARRR, plan en cours

## Project Overview

Le repo héberge **deux apps cohabitantes** dans un seul Next.js 16 :

| App        | Domaine            | Route group         | Usage                                            |
| ---------- | ------------------ | ------------------- | ------------------------------------------------ |
| **CoList** | `colist.fr`        | `src/app/(colist)/` | Coordination de repas (Noël, événements famille) |
| **Sortie** | `sortie.colist.fr` | `src/app/(sortie)/` | Organisation de sorties culturelles entre amis   |

Routing par hostname dans `src/proxy.ts` (Next 16 utilise `proxy.ts` à la place de `middleware.ts`). DB PostgreSQL et auth Better Auth partagées (cookies cross-subdomain sur `.colist.fr`).

**Stack** : Next.js 16 (App Router, Turbo) · React 19 · TypeScript 5.4 · PostgreSQL via Drizzle ORM 0.45 · Better Auth 1.6 · next-intl 4 (12 locales — CoList only) · Tailwind CSS 3.4 · Shadcn/Radix · Framer Motion 12 · AI SDK 6 (Google/Mistral/OpenAI/OpenRouter) · Vercel Blob · Upstash Redis · Vitest 4 · Playwright.

## Commands

```bash
# Dev
npm run dev              # Next dev avec Turbo

# Quality
npm run lint             # ESLint
npm run lint:fix
npm run format           # Prettier
npm run typecheck        # tsc --noEmit
npm run check            # lint + format:check + typecheck

# Tests
npm test                 # Vitest run (unit, *.test.ts à côté du source)
npm run test:watch
npm run test:e2e         # Playwright
npm run test:e2e:install

# Database (Drizzle)
npm run db:generate      # Génère migration depuis schema
npm run db:migrate       # Applique migrations (auto en build Vercel)
npm run db:push          # ⚠️ dev-only, ne marque pas la table de tracking
npm run db:studio
npm run db:seed
npm run db:cleanup-sessions
npm run db:audit-blob-orphans
```

## Architecture

```
src/
├── app/
│   ├── (colist)/[locale]/    # CoList — i18n routing (12 locales, default fr)
│   ├── (sortie)/             # Sortie — FR-only, pas de [locale]
│   ├── actions/              # Server Actions CoList (mutations)
│   └── api/                  # Routes (cron, AI parsers, auth)
├── features/                 # Modules domaine (components + hooks + lib + queries + actions)
│   ├── events/ meals/ items/ people/ services/ ingredients/   # CoList
│   └── sortie/                                                # Sortie
│       ├── components/  hooks/  lib/  queries/  actions/
├── components/ui/            # Primitives Shadcn/Radix génériques
├── lib/                      # Core utils partagés
│   ├── db.ts                # Drizzle instance
│   ├── auth.ts auth-config.ts auth-client.ts
│   ├── sanitize.ts          # XSS protection
│   ├── action-utils.ts      # createSafeAction wrapper
│   ├── analytics.ts         # ⚠️ tracking CoList legacy (à isoler — voir ANALYTICS_AUDIT.md)
│   └── umami.ts redis.ts ai.ts ...
├── proxy.ts                  # Next 16 — routing hostname (colist vs sortie)
├── hooks/  i18n/  data/
drizzle/
├── schema.ts                 # Schéma DB (CoList + table dédiée `sortie`)
└── migrations/               # Auto-générées, appliquées en build via vercel.json
messages/                     # 12 locales (CoList uniquement)
```

## Key Patterns

**Server Actions** : toutes les mutations utilisent `createSafeAction(schema, handler)` avec validation Zod. Schémas CoList dans `src/app/actions/schemas.ts` ; schémas Sortie dans `src/features/sortie/actions/`.

**Sanitization** (`src/lib/sanitize.ts`) — toujours appliquer avant insertion DB :

- `sanitizeText()` — descriptions
- `sanitizeStrictText()` — noms/titres
- `sanitizeSlug()` — slugs URL
- `sanitizeEmoji()` — emoji unique

**Boundary client/serveur** : Server Components par défaut (data fetching, auth checks). `"use client"` uniquement sur les feuilles interactives.

**i18n** :

- **CoList** : tout texte via `useTranslations()` + clés dans **les 12 fichiers** `messages/{locale}.json` (sinon `MISSING_MESSAGE` au build).
- **Sortie** : **FR-only, strings hardcodées, pas de useTranslations**. Décision produit. Ne pas proposer de migration i18n.

**Sortie — design system "Acid Cabinet"** : palette `ink-*` **inversée** (ink-700 = crème `#F5F2EB`, surface-50 = noir `#0F0F0F`). Tokens scopés `.theme-sortie` dans `src/app/sortie.css`. Avant de toucher un composant Sortie, vérifier `tailwind.config.ts` (~ll. 67-78) et cross-référencer un composant existant.

**Sortie — sécurité argent/auth/IBAN** : tout changement sur paiement, dette, IBAN, magic-link, ou step-up code → invoquer un sub-agent sécurité avant d'écrire du code. Spec a tranché des écarts validés (magic link anonyme 24h, short_id 8 chars sans ambigus, step-up 6 chiffres pour actions financières).

## Code Standards

- **No `any`** — utiliser `unknown` ou des génériques.
- **Mobile-first** — touch targets ≥ 44 px (`h-11`), `<button type="button">` sémantique partout.
- **A11y** — `aria-label` sur boutons icon-only, `focus-visible` rings (ring-accent/50), labels sur inputs.
- **Transitions** — `duration-300` standard ; pour Sortie utiliser le motion vocabulary (`motion-tap`/`standard`/`emphasized`) — voir `DESIGN_SYSTEM.md`.
- **Reduced motion** — toujours préfixer `motion-safe:` (WCAG 2.3.3).
- **Tests** — `*.test.ts` co-localisés à côté du source.

## Workflow attendu

**Avant d'implémenter du non-trivial** (refactor multi-PRs, refonte UI, plan analytics, sécu) :

1. **Reviews croisées** : 2-3 sub-agents en parallèle (UX architect + senior React + devil's advocate / sécu / produit) avec rôles complémentaires.
2. **Validation context7** sur tout pattern récent (React 19 `useTransition`, Next 16 cache components, framer-motion `layoutId`, AI SDK v6, Drizzle, Better Auth) — ne pas asserter de mémoire.
3. **Synthèse opinionated** : présenter convergences + divergences + reco tranchée. Ne pas renvoyer un QCM à l'owner ; trancher via experts. L'owner peut overrider.
4. **Pas de belt-and-suspenders** : si le mécanisme primaire (split de sites, séparation env) couvre le cas, ne pas empiler tag/flag/guard redondant.

**Pendant l'implémentation** :

- **Commits fréquents et atomiques** sur tâches multi-étapes (chaque step qui typecheck = 1 commit).
- Ne pas anticiper les phases suivantes ; respecter strictement le scope courant.

**En fin de phase** :

- Invoquer un sub-agent simplification pour traquer code mort, abstractions prématurées, error handling défensif inutile.

## Database Changes

1. Modifier `drizzle/schema.ts`
2. `npm run db:generate` — review la migration dans `drizzle/migrations/`
3. Commit la migration **avec** le code applicatif qui la consomme
4. Push → `vercel.json` exécute `db:migrate && next build` automatiquement (preview ET prod sur la même `DATABASE_URL` Neon — décision validée)

**Ne pas utiliser `db:push` sur la prod** : il ne marque pas `drizzle.__drizzle_migrations` → désync future.

## Sortie — points d'attention persistants

- **Silent users mesurés 2026-05-05** : 5 rows total, 0 > 90j, inflow 0.17/jour. Non actionnable au volume actuel. Surveiller via `npx tsx scripts/db/count-silent-users.ts`. Plan de cleanup en standby (cf. memory) jusqu'à seuil 1k stock OU 50/jour inflow OU 100 RSVP/jour Umami.
- **PWA/native abandonnés** (2026-05-04) — ne pas re-proposer sans signal explicite.
- **Wizard `/sortie/nouvelle`** — refactor multi-PRs en checkpoint ; `index.tsx` ~2200 lignes, plan PR4-5 conditionné aux métriques Umami.
- **Analytics** — Phase 3 PR1 en cours ; voir `ANALYTICS_AUDIT.md` §9 pour le cadre révisé (AAARRR, seuils absolus < 200 sessions/sem, segmentation device).

---
> Source: [yogz/Nawel](https://github.com/yogz/Nawel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
