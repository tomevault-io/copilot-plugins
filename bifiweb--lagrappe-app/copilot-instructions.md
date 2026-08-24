## lagrappe-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

- **Next.js 14** (App Router, TypeScript) — `src/app/`
- **Supabase** — Auth (Google OAuth), PostgreSQL, Realtime
- **Tailwind CSS** + **Framer Motion** — styling/animations
- **Zustand** — client state (tasting draft, session)
- **Shopify Storefront API** — wine product links

## Commandes

```bash
npm run dev          # dev server → http://localhost:3000
npm run build        # build production
npm run lint         # ESLint

npm run db:push      # appliquer les migrations Supabase
npm run db:reset     # reset complet de la base locale
npm run db:types     # regénérer src/types/supabase.ts depuis le schéma local
```

> Note : `next.config.js` désactive TypeScript et ESLint errors au build (`ignoreBuildErrors: true`).

## Architecture

### Pages (`src/app/`)

```
auth/login/           → connexion Google OAuth
auth/callback/        → échange du code OAuth (client-side, pas de route serveur)
app/
  dashboard/          → accueil joueur
  project/[slug]/     → page projet (liste des sessions)
  session/[id]/       → lobby + vote chef + chifoumi tiebreak
  session/[id]/reveal → écran de résultats après dégustation
  tasting/[id]/       → fiche dégustation 5 étapes (vue/nez/bouche/notes/devinette)
  cave/               → hub : Vins dégustés (jeu + ratings manuels fusionnés)
  cave/pepites/       → cave à pépites (vins du catalogue, notation inline)
  profile/            → profil joueur
  quiz/[id]/          → quiz éducatifs
  start/[slug]/       → entrée par QR code / lien partagé (redirige vers login si non auth)
  admin/catalog/      → CRUD du catalogue cave à pépites (admin only)
```

Toutes les pages sont `'use client'`. L'auth est vérifiée manuellement via `supabase.auth.getUser()` en début de chaque page, pas via middleware.

### Flux de jeu

```
Dashboard → Project → Session (lobby)
  → Vote chef → Chifoumi si égalité
  → Tasting (5 étapes)
  → Waiting reveal → Reveal + Scores
```

Le statut d'une session est géré via `sessions.status` en BDD : `lobby → voting → tasting → waiting_reveal → revealed → finished`. Les transitions sont synchronisées par Supabase Realtime.

### État client (`src/store/`)

- `useTastingStore` — draft de la fiche en cours (persiste en localStorage via `zustand/persist`, clé `lagrappe-tasting`). Se reset à la soumission.
- `useSessionStore` — liste des joueurs et statut session (pas persisté).

### Realtime (`src/hooks/useRealtime.ts`)

- `useSessionRealtime(sessionId, callbacks)` — écoute `sessions` + `session_players` via postgres_changes.
- `useVotesRealtime(sessionId, onVoteAdded)` — écoute les votes chef.
- Le chifoumi tiebreak utilise un channel Broadcast Supabase séparé (`session_chifoumi:{sessionId}`).

### Supabase

- `src/lib/supabase/client.ts` — `createBrowserClient` (usage côté client)
- `src/lib/supabase/server.ts` — `createServerClient` avec cookies Next.js (usage dans Server Components / Route Handlers)
- Pas de middleware Supabase (supprimé car il causait des conflits avec le callback OAuth).

### Base de données

Migrations numérotées dans `supabase/migrations/`. Tables principales :

| Table | Rôle |
|-------|------|
| `profiles` | Utilisateurs (role: `admin`\|`player`) |
| `projects` | Projets de dégustation (ex: Swiss Wine Challenge) |
| `wines` | Vins liés à un projet (numérotés, révélés progressivement) |
| `grappiste_notes` | Notes officielles par vin (robe, arômes, prix exact…) |
| `sessions` | Une dégustation d'un vin (status machine d'état) |
| `session_players` | Joueurs dans une session, points, pseudo, avatar |
| `evenings` | Soirée continue (plusieurs sessions enchaînées) |
| `tastings` | Fiche remplie par un joueur |
| `votes` | Votes pour le chef de dégustation |
| `catalog_wines` | Vins de la cave à pépites (indépendant du jeu) |
| `cave_ratings` | Notes manuelles d'un joueur sur un vin du catalogue |

Le scoring officiel est calculé en SQL (fonction `calculate_session_scores`). `src/lib/scoring/index.ts` fournit un calcul client-side pour prévisualisation uniquement.

### Types (`src/types/index.ts`)

Source de vérité pour tous les types TypeScript et les constantes :
- `WINE_CONTENT` — listes de robes/arômes/bouche/accords/cépages/régions par type de vin
- `SCORING` — barème des points par critère
- `PRIX_OPTIONS` / `PRICE_RANGES` — tranches de prix
- `ELEVAGE_OPTIONS` — options d'élevage

### Accès par lien / QR code

`/start/[slug]` — point d'entrée public. Redirige vers `/app/project/[slug]` si connecté, sinon vers login avec `?redirect=` pour revenir après auth.

### WineMode (`src/store/wineMode.tsx`, `src/components/WineMode.tsx`)

Fonctionnalité Easter egg / mode soirée. Wrappé globalement dans le `RootLayout` via `WineModeProvider`.

## Variables d'environnement

Voir `.env.example`. Clés requises :
- `NEXT_PUBLIC_SUPABASE_URL` + `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY` (serveur uniquement)
- `NEXT_PUBLIC_SHOPIFY_DOMAIN` + `NEXT_PUBLIC_SHOPIFY_STOREFRONT_TOKEN`
- `CLAUDE_API_KEY` (tiebreak pseudo)
- `NEXT_PUBLIC_APP_URL`

## Migrations

Pour appliquer une nouvelle migration :
1. Créer `supabase/migrations/NNN_description.sql`
2. Appliquer via `npm run db:push` (ou SQL Editor Supabase en prod)
3. Regénérer les types : `npm run db:types`

## Rôles

- `admin` — gestion projets/vins/catalogue, accès `/app/admin/*`, peut révéler les vins
- `player` — accès jeu, cave, quiz

Promouvoir un admin :
```sql
update public.profiles set role = 'admin' where email = 'email@exemple.com';
```

---
> Source: [bifiweb/lagrappe-app](https://github.com/bifiweb/lagrappe-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
