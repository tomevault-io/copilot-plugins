## recettes-web

> > AI coding assistant instructions for recettes-web

# AGENTS.md

> AI coding assistant instructions for recettes-web

> Généré par l'init contexte le 2026-07-04 — chaque fait ci-dessous a été sourcé depuis le code à cette date.
> Relancer l'init contexte après un changement structurel (nouveau repo, nouveau topic, changement de stack ou d'infra).

## Project Overview

`recettes-web` est l'interface utilisateur React/TypeScript du carnet de recettes.
Elle consomme `recettes-api` via une couche HTTP typée et affiche une grille de recettes
avec recherche instantanée (debounce 250 ms), ajout et suppression.
Ce repo sert aussi de cas de test Forge (`package.json:4`).

- **Domain**: Gestion de recettes de cuisine (UI)
- **Type**: SPA (Single Page Application)
- **Upstream**: `recettes-api` — `GET/POST/PUT/DELETE /api/recipes`
- **Downstream**: aucun

## Tech Stack

| Layer | Technology | Version |
|-------|------------|---------|
| Language | TypeScript | 5.5.3 (`package.json:19`) |
| UI Framework | React | 18.3.1 (`package.json:12`) |
| Build / Dev server | Vite | 5.4.0 (`package.json:18`) |
| Vite plugin | @vitejs/plugin-react | 4.3.1 (`package.json:17`) |
| Module system | ES Modules (`"type":"module"`) | — (`package.json:5`) |
| State management | React hooks uniquement (`useState`, `useCallback`) | — (`src/App.tsx:1`) |
| Routing | Aucun router — vue unique | — |
| Design system | CSS custom (`src/styles.css`) | — |
| Testing | *Non trouvé au scan.* | — |

## Project Structure

```
recettes-web/
├── src/
│   ├── App.tsx              # Shell : state, debounce recherche 250 ms, handlers CRUD
│   ├── api.ts               # Client HTTP typé pour recettes-api
│   ├── main.tsx             # Entry point (ReactDOM.createRoot + StrictMode)
│   ├── styles.css           # CSS global (.app, .topbar, .card, .grid, .tag…)
│   ├── vite-env.d.ts        # Types d'env Vite
│   └── components/
│       ├── RecipeCard.tsx   # Affiche une recette (titre, desc, meta, ingrédients, tags)
│       └── RecipeForm.tsx   # Formulaire "Ajouter une recette" (champs → tableaux)
├── index.html
├── vite.config.ts           # Dev port 5180, proxy /api → localhost:4000
├── tsconfig.json            # strict, noUnusedLocals, noUnusedParameters
└── package.json
```

## Critical Files

| File | Purpose |
|------|---------|
| `src/api.ts` | Client HTTP unique ; tous les `fetch` passent ici ; proxifié via `/api` |
| `src/App.tsx` | Composant racine ; détient tout l'état ; debounce recherche 250 ms |
| `vite.config.ts` | Port dev 5180, proxy `/api` → `http://localhost:4000` |
| `tsconfig.json` | Mode strict TypeScript + `noUnusedLocals/Parameters` enforced |

## Commands

```bash
npm install
npm run dev      # Vite dev server sur http://localhost:5180 (proxy /api → :4000)
npm run build    # tsc -b && vite build → dist/
npm run preview  # Servir dist/ localement
```

## Frontend Specifics

- **UI framework**: React 18, composants fonctionnels uniquement
- **Design system**: classes CSS custom (`.card`, `.grid`, `.topbar`, `.tag`…) — pas de bibliothèque UI
- **State management**: `useState` + `useCallback` dans `App.tsx` ; pas de Redux/Zustand/Context
- **Routing**: Vue unique, pas de react-router
- **API layer**: `src/api.ts` — tous les appels proxifiés via Vite vers `recettes-api` sur le port 4000
- **E2E / Tests**: *Non trouvé au scan.* — pas de script `test` dans `package.json`
- **Target API**: la variable d'env `VITE_API_TARGET` surcharge la cible du proxy (`vite.config.ts:12`)

## Coding Conventions

### Always
- Typage complet : `Recipe` et `RecipeInput` définis dans `src/api.ts` et importés dans tout le projet
- TypeScript strict (`tsconfig.json:13`) : corriger tous les warnings `noUnusedLocals` avant de committer
- Un composant par fichier sous `src/components/`
- Erreurs surfacées dans l'UI via `setError()` → `<div className="banner error">` (`src/App.tsx:70`)

### Never
- Mettre des appels `fetch` en dehors de `src/api.ts`
- Utiliser `any` (mode strict TS enforced)
- Ajouter des bibliothèques de state global sans discussion (approche intentionnellement minimale)

### Ask First
- Ajouter un router (vue unique par conception)
- Modifier les interfaces `Recipe` ou `RecipeInput` (contrat avec recettes-api)
- Ajouter un framework CSS ou une bibliothèque de composants

## Gotchas

- Aucun test configuré — `npm test` échouerait ; discuter avant d'ajouter un test runner
- `recettes-api` doit tourner sur le port 4000 avant le dev server, sinon toutes les requêtes API échouent
- `VITE_API_TARGET` doit inclure le schéma (`http://...`), pas seulement le hostname
- Le frontend exporte `updateRecipe` dans `src/api.ts` mais l'UI ne l'utilise pas encore (`src/api.ts:43`)

---
> Source: [Karindath/recettes-web](https://github.com/Karindath/recettes-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
