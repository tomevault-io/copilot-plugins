## omora

> - `docs/ARCHITECTURE.md` — diagrammes Mermaid complets : services, pipeline ingestion, flow recherche (séquence SSE), RRF, 3 stores, multi-tenant, déploiement

# CLAUDE.md — Moteur de recherche IA Immobilier

## Documentation du projet

- `docs/ARCHITECTURE.md` — diagrammes Mermaid complets : services, pipeline ingestion, flow recherche (séquence SSE), RRF, 3 stores, multi-tenant, déploiement
- `docs/DATA_MODEL.md` — modèle de données, ERD, règles de design du schéma, pattern adaptateur, spec WordPress complète
- `docs/ROADMAP.md` — 6 sprints détaillés jour par jour avec Test Gates
- `docs/TESTING.md` — stratégie de test 4 niveaux, organisation par package
- `docs/SPRINT_0.md` — instructions de démarrage immédiat
- `docs/assets/` — schémas SVG visuels (pipeline, flow recherche, enrichissement)
- `infrastructure/migrations/001_initial_schema.sql` — schéma SQL exécutable complet
- `packages/core/src/property.types.ts` — types TypeScript + exemple d'adaptateur

## Vision du projet

Moteur de recherche conversationnel IA embarquable sur les sites d'agences immobilières (SaaS multi-tenant). L'utilisateur décrit son projet en langage naturel ("maison familiale, 2 enfants, un chien, du soleil, 45 min de Bordeaux, 350k max"), l'agent IA extrait les critères, effectue une recherche hybride et retourne des biens pertinents avec argumentation, comme un vrai agent immobilier.

**Les deux exigences non-négociables : rapidité (ressenti < 1s grâce au streaming) et pertinence.**

## Architecture

### Principe fondamental

Tout le travail lourd se fait à l'INGESTION, jamais à la recherche. Quand l'utilisateur cherche, tout est déjà calculé, enrichi et indexé.

### Les 3 stores (un bien vit aux 3 endroits)

- **PostgreSQL (Supabase)** : source de vérité complète. Schéma normalisé ~120 colonnes. PostGIS pour le géo.
- **Typesense** : index full-text BM25 allégé. Filtres stricts (budget, pièces, ville, tags). Répond < 10ms.
- **Qdrant** : vecteurs 1536 dim (text-embedding-3-small). Recherche sémantique cosine. Répond < 40ms.
- **Redis** : cache requêtes (TTL 5min) + sessions conversation.

### Flow de recherche (latence cible P95 < 1.2s)

1. Message utilisateur → widget → API (SSE ouvert)
2. Claude Haiku extrait les critères → JSON `{must_have, nice_to_have, rayon_geo}` (~300ms)
3. Typesense (filtres stricts) + Qdrant (similarité sémantique) en PARALLÈLE (~40ms)
4. Fusion RRF (Reciprocal Rank Fusion, k=60) + business rules (exclusion must_have manquant, bonus nice_to_have) → top 5 (~5ms)
5. Claude Sonnet génère la réponse en streaming avec les 5 biens en contexte compact (< 500 tokens de biens)

### Pipeline d'ingestion

1. Source (WordPress REST API en priorité V1) → adaptateur normalise vers le schéma unifié
2. Géocodage via API Adresse data.gouv.fr si pas de lat/lng
3. Enrichissement géo en parallèle (~200ms) : Overpass OSM (écoles, commerces, transports), Open-Meteo (ensoleillement), Géoportail/BRGM (risques)
4. 1 appel Claude : tags lifestyle, profils acheteurs, points forts, texte synthétique pour embedding
5. Dispatch parallèle vers les 3 stores

**Optimisation clé : `source_hash`** (MD5 du payload brut). Si inchangé au sync suivant → zéro retraitement (pas d'appel LLM, pas d'appel géo, pas de réindexation). ~95% d'économie sur les syncs quotidiens.

## Structure du monorepo (pnpm workspaces)

```
immo-ai-search/
├── apps/
│   ├── api/              # Backend Express + TypeScript (SSE streaming)
│   ├── widget/           # React 19 Web Component (Vite, Shadow DOM, < 80kb)
│   └── admin/            # Back-office agences (V1 minimal)
├── packages/
│   ├── core/             # Types partagés, schémas Zod, enums
│   ├── db/               # Client postgres.js + runner migrations SQL
│   ├── search/           # Clients Typesense + Qdrant + algorithme RRF
│   ├── ingestion/        # Pipeline orchestrateur + adaptateurs sources
│   ├── ai/               # Wrappers Claude API + OpenAI embeddings
│   └── geo/              # Clients Overpass, Open-Meteo, Géoportail
├── infrastructure/
│   ├── docker/           # docker-compose.yml dev + prod
│   ├── migrations/       # SQL versionnés (001_initial_schema.sql, ...)
│   └── scripts/          # seed.ts, reset, benchmark
└── .github/workflows/    # CI
```

## Stack technique

- **Runtime** : Node.js 22, TypeScript strict partout, React 19 (widget)
- **DB** : PostgreSQL 16 + PostGIS via Supabase (prod) / Docker local (dev). Driver `postgres.js` — PAS d'ORM, PAS de Prisma (le schéma utilise PostGIS, TEXT[], JSONB, triggers — Prisma gère mal). Migrations SQL manuelles versionnées.
- **Search** : Typesense 27 + Qdrant (les deux self-hosted Docker)
- **IA** : Claude Haiku (extraction critères), Claude Sonnet (agent conversationnel), OpenAI text-embedding-3-small (embeddings)
- **Tests** : Vitest (unit/integration), Testcontainers (vrais Postgres/Redis en test), MSW (mock APIs externes), Supertest (API), React Testing Library (widget), Playwright (E2E), k6 (charge)

## Infrastructure

- **Dev local** : docker-compose (Postgres+PostGIS, Typesense, Qdrant, Redis) + hot reload
- **Prod** : VPS Hostinger (Docker Compose : API + Typesense + Qdrant + Redis + Caddy HTTPS) + Supabase (PostgreSQL, pooler port 6543) + Cloudflare (DNS, cache widget.js)
- Supabase : activer les extensions `postgis`, `pg_trgm`, `unaccent`. Toujours utiliser le pooler (6543), pas la connexion directe (5432).

## Schéma de données — décisions clés

Le schéma complet est dans `infrastructure/migrations/001_initial_schema.sql`. Tables : `agencies` (tenants), `ingestion_sources`, `properties` (~120 colonnes), `property_price_history`, `ingestion_logs`, `conversations`.

Règles de design :

1. **Colonnes scalaires + index dédiés pour tout ce qui est filtré** (has_garden, price, rooms_bedrooms...). JAMAIS de JSONB pour les champs de recherche.
2. **JSONB uniquement** pour : photos, widget_config, raw_payload (debug/replay), ai_lifestyle_match.
3. **Tableaux PostgreSQL natifs + index GIN** pour les tags : tags_lifestyle TEXT[], buyer_profiles TEXT[].
4. **Distances géo en colonnes INTEGER séparées** (dist_school_primary_m, dist_tram_m...) pour les index partiels.
5. **UNIQUE(source_id, external_id)** pour la déduplication par source.

## Multi-source — pattern Adaptateur

Chaque source implémente l'interface `SourceAdapter` (dans packages/ingestion) :

- `normalize(raw)` → Property normalisée (méthode principale)
- `computeHash(raw)` → hash MD5 stable pour déduplication
- `extractExternalIds(batch)` → détection des biens disparus (à archiver)

Ajouter une source = nouvelle classe + enregistrement dans le `SourceAdapterRegistry`. Le schéma DB ne change JAMAIS.

**Source prioritaire V1 : `wordpress_api`** — WP REST API (`/wp-json/wp/v2/{post_type}`). Les biens sont des Custom Post Types, les caractéristiques dans des meta fields (noms variables selon plugin : Houzez, WP Residence, Estatik). Le `field_mapping` est configurable par source avec presets plugins courants. Sync incrémental via `?modified_after=`. Pagination via headers X-WP-Total.

## Multi-tenant

- Une agence = un tenant = une API key unique (widget)
- Isolation STRICTE : toute requête est filtrée par agency_id. Une agence ne doit JAMAIS voir les biens d'une autre. Tests de sécurité dédiés obligatoires.
- Rate limiting par agence via Redis.
- Défense en profondeur : Row Level Security (RLS) PostgreSQL sur les tables tenant — même une requête buggée sans filtre agency_id ne peut pas fuiter de données (migration dédiée, Sprint 0 avec packages/db).
- **Onboarding manuel en V1** : pas d'inscription publique. Le client signe un contrat, l'opérateur crée le tenant (script/CLI `create-agency` : agence + clé API + config source), configure l'ingestion et transmet le snippet widget. Le self-service complet est V2.

## Conventions de code

- TypeScript strict, pas de `any` non justifié
- Validation Zod sur toutes les entrées API
- Erreurs : jamais de crash sur des données manquantes dans le pipeline d'ingestion — log warning + continuer
- Les appels externes (géo, LLM) ont toujours timeout + fallback gracieux
- Logs structurés Pino : chaque requête logge tenant, latence par étape, tokens LLM
- Commits conventionnels (feat:, fix:, test:, chore:)

## Tests — Test Gates obligatoires

Chaque sprint se termine par un Test Gate 100% vert avant d'ouvrir le suivant :

1. **Unitaires** (Vitest) — logique pure, < 30s, à chaque commit
2. **Intégration** (Vitest + Testcontainers + MSW) — vrais services, APIs mockées, à chaque PR
3. **Fonctionnels** (Supertest) — endpoints HTTP réels
4. **E2E** (Playwright) — flow utilisateur complet sur staging

Règles : coverage ≥ 80% (ne peut jamais baisser), un bug trouvé = test reproduisant le bug AVANT le fix, aucun test flaky toléré (réparé < 24h ou réécrit), jamais de .skip.

Scripts : `pnpm test:unit`, `pnpm test:integration`, `pnpm test:functional`, `pnpm test:e2e`, `pnpm test:gate` (tout).

## Roadmap — 6 sprints de 2 semaines

| Sprint     | Objectif                              | Démo de validation                           |
| ---------- | ------------------------------------- | -------------------------------------------- |
| 0 (S1-2)   | Setup + pipeline ingestion WordPress  | Bien WordPress → 3 stores enrichi            |
| 1 (S3-4)   | Recherche hybride + RRF               | POST /search → top 5 pertinent < 100ms       |
| 2 (S5-6)   | Agent conversationnel + streaming SSE | Conversation complète < 1.2s P95             |
| 3 (S7-8)   | Widget embarquable + multi-tenant     | Widget live dans page HTML, isolation testée |
| 4 (S9-10)  | Déploiement prod VPS + agence pilote  | Prod HTTPS, monitoring, agence réelle        |
| 5 (S11-12) | Facturation + outillage onboarding    | Onboarding opérateur complet < 30 min        |

## Périmètre V1 — discipline stricte

DANS la V1 : ingestion WordPress/CSV/manuel, enrichissement géo+LLM, recherche hybride, agent streaming, widget 2 lignes, multi-tenant, facturation Stripe (invoicing, 2 plans), outillage onboarding opérateur (CLI), back-office minimal orienté opérateur.

HORS V1 (ne pas développer même si tentant) : portail self-service + checkout Stripe (V2), adaptateurs SeLoger/Apimo (V1.1), dashboard analytics (V1.1), carte interactive (V1.1), alertes email (V1.2), estimation IA (V2), multilingue (V2).

Règle : une fonctionnalité entre en V1 seulement si elle est indispensable pour qu'une agence PAIE.

## Coûts cibles

- ~0.004€ / conversation (Haiku extraction + Sonnet réponse)
- Embedding : ~0.10€ / 1000 biens ingérés
- Infra : ~40-50€/mois (Supabase Pro + APIs) hors VPS

## Latences cibles (à respecter, mesurées par k6)

- Typesense : < 10ms
- Qdrant : < 40ms
- RRF : < 5ms
- Extraction critères : < 500ms
- Premier token streaming : < 1s
- P95 bout en bout : < 1.2s
- Pipeline ingestion : > 100 biens/min

---
> Source: [HugoBld/omora](https://github.com/HugoBld/omora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
