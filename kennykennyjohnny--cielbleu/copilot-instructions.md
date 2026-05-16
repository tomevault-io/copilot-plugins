## cielbleu

> Web app Next.js qui montre les terrasses ensoleillées à Paris en temps réel.

# CielBleu — Instructions pour Claude Code

## Projet
Web app Next.js qui montre les terrasses ensoleillées à Paris en temps réel.
- Domaine test : hopleon.fr — Domaine prod : cielbleu.fr
- Supabase project : `cielbleu` (région eu-west-3)
- Hébergement : Vercel
- MVP cible : fin avril 2026

## Stack
- Next.js 15 App Router + TypeScript strict
- Tailwind CSS v4 (config inline dans styles/globals.css via `@theme`)
- Supabase (PostgreSQL + PostGIS + Auth + Storage)
- Mapbox GL JS via react-map-gl
- SunCalc.js pour le calcul de position du soleil
- Google Places API (server-side uniquement)
- Lucide React pour les icônes

## Structure
```
app/              # App Router (layout.tsx, page.tsx, api/...)
components/Map/   # MapView, SunPin, Filters, PlacePreview
lib/              # supabase, sunScore, googlePlaces, weather, suncalc
data/             # buildings.json (hauteurs OSM)
scripts/          # importPlaces.ts (tsx)
styles/           # globals.css (Tailwind v4 + @theme)
supabase/         # schema.sql
types/            # index.ts (Place, SunScore, Building, ...)
```

## Conventions
- Composants : PascalCase, fichiers `.tsx`
- Fonctions/hooks : camelCase
- Types : PascalCase, dans `types/index.ts`
- Imports absolus avec `@/` (jamais de `../../`)
- Pas de `any`, pas de `.then()` (toujours async/await)
- Mobile-first, light mode uniquement

## Palette (CSS variables dans globals.css)
- `--color-soleil` `#FFBE0B` — accent jaune (score plein soleil)
- `--color-ciel` `#3A86FF` — bleu ciel (focus, liens)
- `--color-nuit` `#1B2838` — texte principal
- `--color-creme` `#FFFDF7` — fond
- `--color-corail` `#FF6B6B` — alerte/CTA secondaire
- `--color-gris` `#8D99AE` — texte secondaire / score faible

Classes Tailwind utilisables directement : `bg-soleil`, `text-nuit`, `text-ciel`, `bg-creme`, etc.

## Typographie
- `font-playfair` → titres h1/h2 + logo "CielBleu"
- `font-outfit` → tout le reste (UI, scores, adresses)

Variables injectées par `next/font` dans `app/layout.tsx`.

## APIs
- **Google Places** : UNIQUEMENT côté serveur (route API ou script). Jamais dans un composant client.
- **Mapbox** : token public `NEXT_PUBLIC_MAPBOX_TOKEN`
- **Supabase** : `lib/supabase.ts` exporte le client browser. Pour le service_role (scripts), utiliser `SUPABASE_SERVICE_ROLE_KEY` côté Node uniquement.
- **OpenWeatherMap** : météo Paris pour pondérer les scores soleil — server-side uniquement.

## Scope V1 (en cours)
1. ✅ Carte Mapbox plein écran style custom palette CielBleu
2. ⏳ Import + affichage de tous les bars Paris (Google Places → Supabase)
3. ⏳ Score soleil sur chaque marqueur (algo SunCalc + bâtiments OSM)
4. ⏳ Panel terrasse au clic : photo + score + timeline + vue 3D SVG + infos

## Hors V1
- Auth utilisateur (Supabase Auth) → V2
- Confirmations communautaires "j'y suis" → V2
- Filtres avancés (terrasse couverte, fumeur, etc.) → V2
- Dashboard restaurateur → V3

## Commandes
- `npm run dev` → dev local sur localhost:3000
- `npm run typecheck` → vérif types sans build
- `npm run lint` → ESLint
- `npm run import:places` → import Google Places (nécessite SUPABASE_SERVICE_ROLE_KEY + GOOGLE_PLACES_API_KEY)
- `npx vercel` → déploiement preview Vercel

## Notes importantes
- Tailwind v4 : la config est dans `styles/globals.css` via `@theme`, pas dans un `tailwind.config.ts`.
- Le projet n'a PAS de `src/` — tout est à la racine (app/, components/, lib/, etc.).
- Le path alias `@/*` mappe la racine du projet, donc `@/components/Map/MapView` = `./components/Map/MapView`.

---
> Source: [kennykennyjohnny/Cielbleu](https://github.com/kennykennyjohnny/Cielbleu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
