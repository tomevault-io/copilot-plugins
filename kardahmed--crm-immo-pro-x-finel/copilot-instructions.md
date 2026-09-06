## crm-immo-pro-x-finel

> SaaS CRM immobilier multi-tenant. Gestion complète du cycle de vente immobilier : leads, clients, biens, transactions, équipes. Chaque workspace (tenant) est isolé avec ses propres données, utilisateurs et configurations.

# CLAUDE.md — CRM IMMO PRO X

## Projet

SaaS CRM immobilier multi-tenant. Gestion complète du cycle de vente immobilier : leads, clients, biens, transactions, équipes. Chaque workspace (tenant) est isolé avec ses propres données, utilisateurs et configurations.

## Stack technique

| Couche | Technologie |
|---|---|
| Framework | Next.js 14 (App Router) |
| UI | React, Tailwind CSS, shadcn/ui |
| ORM | Prisma |
| Base de données | Supabase PostgreSQL |
| Auth | Clerk |
| Temps réel | Supabase Realtime |
| Cartographie | Google Maps API |
| Scraping | Puppeteer |
| IA | API Anthropic (Claude) |
| Déploiement | Vercel |

## Architecture multi-tenant

- Chaque table métier porte une colonne `tenant_id` (UUID, NOT NULL).
- Row-Level Security (RLS) activé sur PostgreSQL : les policies filtrent par `tenant_id`.
- Le middleware Next.js extrait le `tenant_id` depuis la session Clerk et l'injecte dans le contexte de chaque requête.
- **Aucune requête ne doit s'exécuter sans filtre `tenant_id`.** C'est la règle n-1 de sécurité.

## Types de workspace

| Type | Description |
|---|---|
| `PROMOTION` | Promoteur immobilier — gère des programmes neufs, lots, réservations |
| `AGENCY` | Agence immobilière — gère un portefeuille de biens existants, mandats, visites |

Le type de workspace conditionne les écrans, les champs affichés et certaines règles métier.

## Rôles utilisateur

| Rôle | Permissions |
|---|---|
| `SUPER_ADMIN` | Accès global cross-tenant, gestion plateforme |
| `CEO` | Admin du workspace, accès complet aux données du tenant |
| `SUPERVISOR` | Gère une équipe d'agents, assigne les leads, voit les stats équipe |
| `AGENT` | Gère ses propres clients et transactions |
| `ASSISTANT` | Accès limité en lecture, support administratif |

## Conventions de code

### Nommage

- **Variables / fonctions** : `camelCase`
- **Composants React** : `PascalCase`
- **Fichiers / dossiers** : `kebab-case`
- **Types / Interfaces** : `PascalCase` avec préfixe `I` pour interfaces (ex: `IClient`)

### TypeScript

- Mode `strict` activé dans `tsconfig.json`.
- Pas de `any` — utiliser `unknown` puis narrowing si nécessaire.
- Valider toutes les entrées externes avec **Zod**.

### API

- Routes : `/api/v1/{resource}` (RESTful).
- Toujours vérifier `tenant_id` dans chaque handler API.
- Retourner des réponses JSON normalisées : `{ success, data, error }`.

### Structure des fichiers

```
src/
  app/              # App Router Next.js (pages, layouts, routes API)
  components/       # Composants React réutilisables
    ui/             # Composants shadcn/ui
  lib/              # Utilitaires, clients (prisma, supabase, clerk)
  hooks/            # Custom React hooks
  types/            # Types TypeScript partagés
  services/         # Logique métier
  middleware.ts     # Middleware Next.js (auth + tenant isolation)
prisma/
  schema.prisma     # Schéma Prisma
```

## Regles metier critiques

### Attribution des clients

- **1 client = 1 agent** : un client ne peut etre assigne qu'a un seul agent a la fois.
- Le changement d'agent necessite une action explicite du `SUPERVISOR` ou `CEO`.

### Detection de doublons

- La detection se fait par **numero de telephone** (normalise au format international).
- Avant toute creation de client, verifier l'existence du numero dans le tenant.

### Pipeline de vente (9 etapes)

1. `NEW` — Lead entrant
2. `CONTACTED` — Premier contact etabli
3. `QUALIFIED` — Besoin et budget valides
4. `VISIT_SCHEDULED` — Visite programmee
5. `VISITED` — Visite effectuee
6. `NEGOTIATION` — Offre / contre-offre en cours
7. `RESERVED` — Reservation signee
8. `SIGNED` — Compromis / acte signe
9. `CLOSED` — Transaction finalisee

### Automatisations au changement d'etape

- Chaque transition d'etape declenche des **actions automatiques** (notifications, mises a jour, taches).
- Les automatisations sont configurables par workspace.
- Exemples : notification agent a `VISIT_SCHEDULED`, alerte superviseur a `NEGOTIATION`, generation doc a `RESERVED`.

### IA — Validation humaine obligatoire

- Toute suggestion generee par l'IA (scoring, recommandation, redaction) **doit etre validee par un humain** avant application.
- Aucune action automatique basee uniquement sur l'output IA.
- Les suggestions IA sont marquees visuellement comme telles dans l'UI.

### Facebook Leads

- Les leads provenant de Facebook Ads sont **automatiquement assignes au Superviseur** de l'equipe concernee.
- Le Superviseur redistribue ensuite manuellement aux agents.
- Pas d'assignation directe Facebook -> Agent.

## Commandes utiles

```bash
# Developpement
npm run dev              # Serveur de dev Next.js
npm run build            # Build production
npm run lint             # ESLint
npm run type-check       # Verification TypeScript

# Base de donnees
npx prisma generate      # Generer le client Prisma
npx prisma db push       # Pousser le schema vers la DB
npx prisma studio        # Interface visuelle Prisma
npx prisma migrate dev   # Creer une migration

# Tests
npm run test             # Lancer les tests
```

---
> Source: [kardahmed/CRM-IMMO-PRO-X-FINEL-](https://github.com/kardahmed/CRM-IMMO-PRO-X-FINEL-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
