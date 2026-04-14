## sci-manager-renew

> **Type:** MicroSaaS B2B

# AGENT Instructions
# GererSCI - Documentation Agents Codex

## Contexte Projet
**Nom:** GererSCI
**Type:** MicroSaaS B2B  
**Marché:** France (500k+ SCI, marché locatif 1Md€)  
**Cible:** Propriétaires SCI familiales, investisseurs LMNP/nu-locatif  
**Modèle:** Freemium (1 bien) | 19€/mois (5 biens) | 49€/mois (illimité) | Lifetime 299€

## Stack Technique
BACKEND: Python 3.12 | FastAPI 0.115+ | Pydantic v2 | Uvicorn
DATABASE: Supabase PostgreSQL (RLS multi-users)
FRONTEND: Sveltekit 2.x | TypeScript | TailwindCSS | shadcn-svelte
AUTH: Supabase Auth (JWT + RLS)
PAIEMENTS: Stripe (subscriptions + lifetime)
INFRA: Docker + Docker Compose | Nginx reverse proxy
TESTS: pytest 85%+ | Playwright E2E | Lighthouse 95+
DEPLOY: VPS Linux (Ubuntu 22.04) + Let's Encrypt SSL

text

## Arborescence Cible
gerersci/
├── AGENTS.md # Ce fichier
├── ARCHITECTURE.md # Phase 1
├── README.md # Phase 6
├── .env.example
├── docker-compose.yml # Phase 6
├── backend/
│ ├── app/
│ │ ├── main.py
│ │ ├── core/
│ │ │ ├── config.py
│ │ │ └── security.py
│ │ ├── api/
│ │ │ └── v1/
│ │ │ ├── biens.py
│ │ │ ├── loyers.py
│ │ │ ├── quitus.py
│ │ │ └── cerfa.py
│ │ ├── models/ # Pydantic
│ │ ├── schemas/ # DB schemas
│ │ └── services/
│ │ ├── sci_service.py
│ │ └── quitus_service.py
│ ├── tests/
│ │ ├── conftest.py
│ │ ├── test_api/
│ │ └── test_services/
│ ├── Dockerfile
│ └── requirements.txt
├── frontend/
│ ├── src/
│ │ ├── lib/
│ │ │ ├── components/
│ │ │ │ └── ui/ # shadcn-svelte
│ │ │ ├── supabase.ts
│ │ │ ├── api.ts
│ │ │ └── stripe.ts
│ │ ├── routes/
│ │ │ ├── +layout.svelte
│ │ │ ├── +page.svelte # Landing
│ │ │ ├── (auth)/
│ │ │ │ ├── login/
│ │ │ │ └── register/
│ │ │ ├── dashboard/
│ │ │ ├── biens/
│ │ │ ├── loyers/
│ │ │ └── pricing/
│ │ └── app.html
│ ├── Dockerfile
│ ├── package.json
│ └── svelte.config.js
├── supabase/
│ ├── migrations/
│ │ └── 001_init.sql
│ └── seed.sql
└── docker/
└── nginx.conf

text

## Tables Supabase (RLS Obligatoire)
```sql
sci(id, nom, siren, regime_fiscal, created_at)
associes(id, id_sci, user_id, nom, email, part, role)
biens(id, id_sci, adresse, ville, code_postal, type_locatif, loyer_cc, charges, tmi, acquisition_date)
locataires(id, id_bien, nom, email, date_debut, date_fin)
loyers(id, id_bien, id_locataire, date_loyer, montant, statut, quitus_genere)
charges(id, id_bien, type_charge, montant, date_paiement)
fiscalite(id, id_sci, annee, total_revenus, total_charges, resultat_fiscal)
```
API Endpoints Backend
```text
/health                          # Healthcheck
/api/v1/auth/*                   # Supabase Auth
/api/v1/sci                      # CRUD SCI
/api/v1/biens                    # CRUD biens
/api/v1/loyers                   # CRUD loyers
/api/v1/quitus/generate          # Génération PDF quitus
/api/v1/cerfa/2044               # Cerfa 2044 auto
/api/v1/fiscalite/simulate       # Simulateur IR/IS
/api/v1/stripe/webhook           # Webhooks Stripe
```
### Conventions Code
Backend Python
Typage strict (Pydantic models)

Async/await systématique

Rate limiting (slowapi)

CORS strict (origins env var)

Logs structurés (structlog)

Tests pytest avec fixtures

Frontend Sveltekit
TypeScript strict mode

Composants shadcn-svelte

Mobile-first responsive

Dark mode par défaut

PWA-ready (manifest + SW)

SSR pour SEO

Sécurité
RLS Supabase toutes tables

JWT validation backend

CORS whitelist

Headers sécurité (CSP, HSTS)

Stripe webhook signature verify

Secrets via .env (jamais hardcodé)

Design System (Dark Business)
text
PRIMARY: #1e293b (Slate 800)
SECONDARY: #f8fafc (Slate 50)
ACCENT: #3b82f6 (Blue 500)
SUCCESS: #10b981 (Emerald 500)
ERROR: #ef4444 (Red 500)
WARNING: #f59e0b (Amber 500)

FONTS: Inter (system-ui fallback)
RADIUS: 0.5rem (8px)
SPACING: Tailwind scale (4/8/12/16/24/32px)
Checklist Production-Ready (100% obligatoire)
Backend ✓
 Structure modulaire complète

 pytest coverage 85%+

 Bandit security scan clean

 RLS Supabase validé

 Stripe webhooks testés

 Rate limiting configuré

Frontend ✓
 npm run build sans erreur

 Responsive mobile parfait

 shadcn-svelte intégré

 PWA manifest + icons

 Lighthouse 95+ (all)

 SSR fonctionnel

Tests ✓
 pytest 100% PASS

 Playwright E2E PASS

 Coverage 85%+

 Accessibility audit 0 error

Infra ✓
 Dockerfiles optimisés

 docker-compose up -d OK

 Nginx config HTTPS

 .env.example complet

Règles Codex
Plan avant code : Toujours proposer plan 5 étapes

Tests obligatoires : Lance pytest/Playwright (pas de simulation)

Validation checkpoints : Arrête après chaque phase pour review

Commits atomiques : 1 feature = 1 commit clair

Documentation inline : Docstrings Python, JSDoc TypeScript

Phases Workflow
ARCHITECTURE : ARCHITECTURE.md + arborescence

BACKEND : FastAPI + Supabase + tests

FRONTEND : Sveltekit + UI + intégration

STRIPE : Paiements + webhooks

TESTS/AUDIT : E2E + sécurité + Lighthouse

DEPLOY : Docker + Nginx + README

Toujours valider checklist 100% avant phase suivant

Référez-vous à `.github/copilot-instructions.md` pour les guides détaillés de phases et conventions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/radnou) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
