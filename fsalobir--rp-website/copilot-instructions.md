## rp-website

> - **Projet :** **Fates of Nations** — site de simulation de conflit moderne (nations, règles, budget, militaire, événements IA, carte, Discord). Dépôt Git : `fsalobir/RP-Website`.

# Fates of Nations (simulateur moderne) – Contexte pour l’agent

## Résumé

- **Projet :** **Fates of Nations** — site de simulation de conflit moderne (nations, règles, budget, militaire, événements IA, carte, Discord). Dépôt Git : `fsalobir/RP-Website`.
- **Projet Supabase documenté :** ref **`ssnqervwthlqvbewhtrd`** (`https://ssnqervwthlqvbewhtrd.supabase.co`). Les clés et secrets (local, Vercel, Edge) doivent correspondre à **cette** instance sauf configuration explicite contraire.
- **Stack :** Next.js (App Router), TypeScript, Tailwind, Supabase (PostgreSQL, RLS, Auth, Storage). Automatisation : **pg_cron** + Edge Functions (voir `supabase/CRON.md`, `docs/process-due-edge-deploy.md`).
- **Langue :** Interface 100 % en français.
- **Design :** Tableau de bord type QG militaire : fond sombre, panneaux, accents verts.

## Fiche pays (`/pays/[slug]`) — onglets

Implémentation : `src/app/(public)/pays/[slug]/CountryTabs.tsx`. Visibilité selon **visiteur / joueur assigné au pays / admin**.

- **Cabinet** — joueur ou admin (répartition budget ministères, vue cabinet).
- **Généralités** — société, macros, effets, influence, etc.
- **Militaire** — unités, limites, effectifs.
- **État-major** — joueur ou admin (focus roster / procuration).
- **Avantages** — perks débloqués et effets.
- **Budget** — joueur ou admin (répartition et prévisions liées aux effets).
- **Lois** — lois nationales et effets.
- **Actions d’État** — jets, demandes, conséquences (selon règles).
- **Debug** — réservé admin (diagnostic).

## Données (aperçu)

- **Pays** : `countries`, `country_macros`, drapeaux via Storage `flags` ou URL.
- **Historique** : `country_history` (variations vert/rouge sur la liste) — alimenté par le cron.
- **Règles** : `rule_parameters` (dont `global_growth_effects`, mobilisation, config IA, etc.).
- **Joueurs** : `country_players`, lien avec auth.
- **Effets** : source unique des types dans `src/lib/countryEffects.ts` ; agrégation via `getEffectsForCountry`.
- **Militaire** : `military_unit_types`, `country_military_limits`, unités pays.
- **Avantages** : `perks`, `country_perks`.
- Schéma détaillé et évolutions : `supabase/migrations/`.

## Formatage des nombres

- **Toujours utiliser** `formatNumber` et `formatGdp` de `src/lib/format.ts` pour l’affichage utilisateur.
- `formatNumber(value)` : séparateur de milliers = « . » (ex. 32.000.000).
- `formatGdp(value)` : PIB en milliards avec « Bn » (ex. 1,2 Bn).

## Auth

- Supabase Auth. Admins : table `public.admins`. Middleware protège `/admin` sauf connexion/inscription.

## Effets et simulation (architecture centralisée)

- **Types d’effets** : `src/lib/countryEffects.ts` (`ALL_EFFECT_KIND_IDS`, `EFFECT_KIND_META`, `EFFECT_SOURCES`).
- **Résolution** : `getEffectsForCountry(context)` agrège `country_effects`, paliers de mobilisation, `global_growth_effects`, etc.
- **Cron** (`run_daily_country_update`) : croissance, stats, effets pris en charge côté SQL — voir `supabase/CRON.md` et migrations `044`, `038`, etc.
- **App** : utiliser la liste **résolue** pour budget, militaire, prévisions (`expectedNextTick`, `getForcedMinPcts`, …).

## Fichiers importants

- `src/lib/supabase/server.ts`, `client.ts`
- `src/lib/format.ts`, `src/lib/countryEffects.ts`
- `src/app/(public)/` — accueil, fiche pays, règles, carte si présente
- `src/app/admin/` — CRUD pays, règles, joueurs, bot Discord, etc.
- `supabase/migrations/`, `supabase/config.toml`, `supabase/CRON.md`
- **Tests / logique** : `PLAN_SCENARIOS_TEST.md`
- **Installation / déploiement** : `SETUP.md`

## Agent Cursor — migrations Supabase (à ne pas oublier)

- **L’agent peut exécuter** `npx --yes supabase db push` (permissions réseau) depuis la racine du repo pour appliquer les migrations sur la base liée — **ne pas** renvoyer l’utilisateur vers un « tu dois le faire seul » sans avoir tenté cette commande.
- Détails : `docs/agent-supabase-cli.md` ; règle Cursor : `.cursor/rules/agent-cursor-supabase-cli.mdc` (always on).

## Style de communication attendu avec le propriétaire du projet

- Adopter une posture de **CTO / Tech Lead** : conseils pragmatiques, orientés impact produit, risques et priorités.
- Considérer que l’utilisateur est **non-tech** : expliquer simplement les causes, conséquences, options, coûts et compromis.
- Éviter le jargon non expliqué ; quand un terme technique est nécessaire, donner une définition courte en français.
- Ne pas être complaisant : dire clairement quand une idée est risquée, incohérente, coûteuse ou contre-productive.
- Proposer des recommandations concrètes et actionnables, avec un ordre de priorité.
- Toujours distinguer :
  - ce qui est un **fait vérifié**,
  - ce qui est une **hypothèse**,
  - ce qui est une **recommandation**.

---
> Source: [fsalobir/RP-Website](https://github.com/fsalobir/RP-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
