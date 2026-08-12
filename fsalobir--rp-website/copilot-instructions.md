## nation-simulator

> Règles du projet Simulateur de nations – stack, langue, données, UI


# Fates of Nations — simulateur de nations (RP-Website)

## Stack
- **Frontend:** Next.js (App Router), TypeScript, Tailwind CSS.
- **Backend / DB / Auth:** Supabase (PostgreSQL, RLS, Auth). Pas de backend séparé ; le client Supabase (côté serveur et client) est utilisé avec RLS.

## Langue
- **Tout le texte visible par l’utilisateur doit être en français** : libellés, boutons, messages, validation, titres de page. Le code et les commentaires peuvent rester en anglais.

## Données (Supabase)
- **Pays** : table `countries` (Généralités, Société, Macros). Drapeaux : upload bucket Storage `flags` ou URL. Table optionnelle `country_macros` (clé-valeur).
- **Historique** : `country_history` (country_id, date, population, gdp, militarism, industry, science, stability) pour variations vert/rouge sur la liste ; rempli par le cron.
- **Règles** : `rule_parameters` (key, value jsonb). Clés notables : `global_growth_effects` (tableau d’effets appliqués à tous les pays), `mobilisation_config`, `mobilisation_level_effects`. Schéma extensible.
- **Effets** : types centralisés dans `@/lib/countryEffects.ts` (ALL_EFFECT_KIND_IDS, EFFECT_KIND_META). Résolution « effets pour un pays » via `getEffectsForCountry(context)` qui agrège country_effects, mobilisation, global_growth_effects (sources extensibles).
- **Militaire** : `military_unit_types` (branch: terre/air/mer), `country_military_limits`.
- **Avantages** : `perks`, `country_perks`.
- **Formatage** : utiliser `formatNumber` et `formatGdp` de `@/lib/format` (séparateur ".", PIB en "X Bn").

## Conventions
- Lecture des données : Server Components, client Supabase serveur (`@/lib/supabase/server`).
- Écriture admin : Client Components ou Server Actions avec client Supabase + RLS. Ne pas exposer la clé service_role dans le front.
- Variables d’environnement : `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` (pas de secrets en dur).
- **UI/UX** : conserver une interface cohérente et lisible (thème type tableau de bord QG militaire : fond sombre, panneaux, accents verts). Utiliser les variables CSS du thème (`var(--background-panel)`, `var(--accent)`, etc.) et les composants existants. États de chargement et listes vides en français.

## Structure des routes
- **Public** : `/` = table des nations (indicateurs + variations si `country_history`) ; `/pays/[slug]` = fiche pays avec onglets **Cabinet, Généralités, Militaire, État-major, Avantages, Budget, Lois, Actions d’État** (et **Debug** admin) — visibilité selon joueur assigné / admin ; `/regles` et autres pages sous `src/app/(public)/`.
- **Admin** : `/admin/connexion`, `/admin/inscription` ; après auth tableau de bord, CRUD pays, règles, joueurs, outils (ex. bot Discord), etc. Middleware protège `/admin` sauf connexion/inscription.
- **Projet Supabase documenté** : ref `ssnqervwthlqvbewhtrd` — variables : `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` ; pas de service_role côté client. Voir `.env.example` et `SETUP.md`.
- **Migrations distantes** : l’agent peut exécuter `npx --yes supabase db push` (voir `.cursor/rules/agent-cursor-supabase-cli.mdc` et `docs/agent-supabase-cli.md`) — ne pas renvoyer l’utilisateur sans avoir tenté.

---
> Source: [fsalobir/RP-Website](https://github.com/fsalobir/RP-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
