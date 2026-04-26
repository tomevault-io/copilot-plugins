## where2fly

> Appli météo parapente pour les sites de vol en France. Affiche les prévisions (vent, direction, rafales, température, pluie) dans un tableau sites × jours avec un code couleur (bon/limite/déconseillé).

# Where2Fly

Appli météo parapente pour les sites de vol en France. Affiche les prévisions (vent, direction, rafales, température, pluie) dans un tableau sites × jours avec un code couleur (bon/limite/déconseillé).

## Stack technique

- **Angular 21** standalone, **zoneless** (pas de Zone.js)
- **Angular Material 21** (Material 3, palette azure)
- **Signals** pour l'état réactif (`signal`, `computed`, `input`, `output`)
- **SCSS** modulaire (`_variables.scss`, `_layout.scss`, `_components.scss`)
- **Vitest** pour les tests
- **API Open-Meteo** (gratuite, sans clé) pour les données météo

## Architecture

```
src/app/
├── components/        # Composants standalone
│   ├── header/        # Logo, navigation semaine, refresh
│   ├── site-selector/ # Chips de sélection (max 3 sites)
│   └── meteo-table/   # Tableau principal + états loading/error/empty
├── models/
│   ├── meteo.models.ts  # Interfaces (Site, SlotData, DayForecast, SiteForecast) + enums (Direction, Condition)
│   └── sites.ts         # Constante SITES[]
└── services/meteo-service/
    ├── abstract-meteo.service.ts  # Interface abstraite
    ├── meteo.service.ts           # Implémentation Open-Meteo
    ├── mock-meteo.service.ts      # Mock (à implémenter)
    └── meteo-utils.ts             # Helpers (direction, conditions, formatage)
```

### Styles

Les styles sont **dans les composants** (`.scss` par composant), jamais globaux. Seul `_variables.scss` reste global pour les design tokens (couleurs, ombres). Les popups Leaflet étant injectées hors du DOM Angular, le `SiteMapComponent` utilise `ViewEncapsulation.None`.

### Patterns

- **Single Responsibility** : chaque composant fait UNE seule chose. Découper dès qu'un composant gère plusieurs responsabilités. Préférer des petits composants réutilisables plutôt qu'un gros composant monolithique
- **Composants** : standalone, inputs via `input()` / `input.required()`, outputs via `output()`
- **Services** : pattern abstract class + implémentation concrète fournie via `providers` dans le composant
- **État** : signals uniquement, pas de NgRx ni de BehaviorSubject
- **Pas de routing** pour le moment (SPA mono-page)

## Conventions de code

- **Langue** : code et noms de variables en anglais, commentaires et labels UI en français
- **Composants** : pas de suffixe `Component` dans le sélecteur (`app-header`, pas `app-header-component`)
- **Imports Material** : directement dans le tableau `imports` du composant, pas de module wrapper
- **Pas de `CommonModule`** sauf si besoin de pipes (`DatePipe` importé seul)
- **Styles dans les composants** : chaque composant a son `.scss`, jamais de styles globaux (sauf `_variables.scss` pour les tokens)

## Commandes

- `npm start` — serveur de dev
- `npm run build` — build production (`npx ng build`)
- `npm test` — tests Vitest

## Git

- Branche `dev` pour le développement
- Branche `main` pour la production
- Commits courts en anglais

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/h2Latte) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
