## svelte-kpsull

> project: svelte-kpsull

# .cursorrules - Projet svelte-kpsull

# --- METADATA ---
# Nom du projet
project: svelte-kpsull

# Stack
stack:
  - svelte
  - tailwindcss
  - daisyui
  - drizzle-orm
  - postgresql (docker)
  - lucia (auth)

# Couleurs / Typography
branding:
  primary: "#ea580c"
  secondary: null
  logo_font: "Asimovian"

# --- INITIALISATION ---
# Actions à exécuter à l'initialisation du workspace
init:
  - git init -b main
  - git remote add origin <remote-url> # à remplacer
  - create: 
      - package.json (svelte + tailwind + daisyui + drizzle + lucia)
      - docker-compose.yml (postgres)
      - .env.example (DATABASE_URL, LUCIA_SECRET, etc.)

# --- ARCHITECTURE ---
# Respect strict du Clean Architecture.
folders:
  - domain/
  - application/
  - infrastructure/
  - ui/
  - infra/drizzle/
  - infra/auth/
  - infra/mailing/

# Règles:
# - Domain: entités, value-objects, interfaces de use-cases
# - Application: implémentations use-cases, orchestration
# - Infrastructure: DB, ORM (Drizzle), adaptateurs (Lucia), providers
# - UI: Svelte components, pages, styles

# --- AUTH (LUCIA) ---
# Lucia doit être intégrée dès l'installation.
# - configurer Lucia pour PostgreSQL via Drizzle
# - rôles: client, creator
# - sessions sécurisées, refresh token si nécessaire
# - endpoints d'auth: /api/auth/login, /api/auth/logout, /api/auth/register
# - stocker user profile minimal dans users table (role, displayName, email, plan, created_at)

# --- DB (DRIZZLE + POSTGRES DOCKER) ---
# - Fournir migration initiale: users, products, subscriptions, orders, order_items, shipments, payments
# - Exemple shortcuts migrations et seeds pour comptes demo

# --- UI / UX RULES (INTRANSIGEANT) ---
ui_rules:
  - style: "Moderne, sobre, espacement unifié, rounded-2xl par défaut"
  - inspiration: "https://www.ly0t.fr/"
  - logo_font: "Asimovian"
  - color: primary -> #ea580c ; no secondary
  - use_daisyui: true # privilégier les composants DaisyUI
  - minimal_native_css: true # n'utiliser du CSS natif que quand Tailwind/DaisyUI ne suffisent pas

# --- RESPONSIVE / BREAKPOINTS ---
responsive:
  - use_tailwind_responsive_classes: true
  - breakpoints: [sm, md, lg, xl, 2xl]
  - ensure: "mobile-first, tests manuels sur mobile/tablette/desktop"

# --- COMPOSANTS UI RÉUTILISABLES (PRIORITÉ MAX) ---
# Tous les composants doivent être configurables par props et éviter toute duplication.
components:
  - Button.svelte:
      props: [variant(primary|ghost|danger|link), size(sm|md|lg), iconBefore, iconAfter, disabled]
      behavior: "gérer classes tailwind/daisy via util function. Un seul fichier pour toutes variantes"
  - Header.svelte:
      props: [user|null]
      contains: [Logo, Nav, AccountMenu, MobileMenu]
  - Footer.svelte:
  - Navbar.svelte:
  - CardProduct.svelte:
      props: [product]
      includes: [image, title, price, tags, actions]
  - Pills.svelte:
      props: [items, activeKeys, onSelect(key)]
  - Badge.svelte:
  - Modal.svelte:
  - Dropdown.svelte:
  - Form/Input.svelte:
      props: [type, label, name, value, error]
  - Toast.svelte:
  - BadgeCommission.svelte: "Affiche 5% lors du checkout seulement"

# Guidelines:
# - Chaque composant doit être atomic, testé visuellement et documenté (README court)
# - Props bien nommés en anglais
# - Exposer des slots quand utile
# - Utiliser des helpers utilitaires pour les classes (classBuilder)

# --- PRODUITS & FILTRES ---
products_list:
  - dynamic_filters: true # update list automatically when filters change
  - no_submit_button: true
  - categories: [Men, Women, Accessories, Clothing]
  - nested_subcategories: true
  - price_filter: true
  - UX: debounce inputs where necessary, but déclencher update automatiquement

# --- PANIER / COMMANDES / LIVRAISON ---
orders:
  - carriers: [MondialRelay, Chronopost, Colissimo...] # adaptateurs
  - shipping_price_added_to_client: true
  - shipment_label_generated_for_seller: true
  - email_tracking: true # notifications client + seller

# --- PAYMENTS ---
payments:
  - commission: 5% # platform fee added at checkout (visible only at checkout)
  - subscriptions: [monthly:100, yearly:1000]

# --- CODE STYLE & GUIDELINES ---
code_style:
  - language: svelte + typescript preferred
  - naming: english, clear, small functions
  - keep_lines_short: true
  - tests: add basic unit tests for core use-cases
  - docs: simple README + component docs
  - production_ready: true # Toutes les fonctionnalités doivent persister en base de données et fonctionner réellement

# --- DEVOPS / DOCKER ---
devops:
  - docker-compose:
      services: [postgres, migrate, app]
  - env_vars: DATABASE_URL, LUCIA_SECRET, SMTP_URL

# --- IA WORKFLOW & BEST PRACTICES ---
ai_workflow:
  - explore_first: true # Avant de développer une fonctionnalité, lire AU MINIMUM 3 fichiers pertinents
  - code_quality:
      - run_after_task: ["pnpm tsc", "pnpm lint"]
      - fix_errors: true # Corriger toutes les erreurs de lint/type avant de considérer une tâche terminée
  - testing_strategy:
      - integration_tests: true # Créer des tests d'intégration pour les fonctionnalités principales
      - add_logs: true # Ajouter des logs stratégiques pour déboguer les fonctionnalités
      - user_feedback_loop: true # Utiliser les retours utilisateur pour corriger les bugs

# --- CHECKLIST À LIVRER ---
deliverables:
  - Repo initialized (svelte-kpsull)
  - Tailwind + DaisyUI configured
  - Drizzle + Postgres docker + migrations initiales
  - Lucia configured and endpoints opérationnels
  - UI library components (Button, Header, Footer, CardProduct, Pills, Modal, Dropdown)
  - Page d'accueil minimal + liste produits + filtres dynamiques
  - Auth flows (login/register client/creator)
  - Creator dashboard minimal CRUD produits
  - Panier simple + checkout stub showing commission & shipping

# NOTES POUR CURSOR
# - Respect strict du pattern Clean Architecture
# - Focus UI: strong emphasis sur composants réutilisables, un seul point d'entrée pour logique partagée
# - Keep it small & simple: "less is better"
# - Prioriser la lisibilité du code Svelte (fonctions courtes, noms explicites)
# - Avant de développer une fonctionnalité, explorer le contexte en lisant au moins 3 fichiers pertinents
# - Toujours vérifier et corriger les erreurs de lint et de type avant de finaliser une tâche
# - Ajouter des logs stratégiques pour faciliter le débogage et améliorer la qualité du code

# FIN

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lyot7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
