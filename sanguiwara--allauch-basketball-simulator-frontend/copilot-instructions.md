## allauch-basketball-simulator-frontend

> Allauch Simulator (frontend) est l'UI du jeu de gestion de basketball.

# Allauch Simulator (Frontend Angular)

## Objectif produit

Allauch Simulator (frontend) est l'UI du jeu de gestion de basketball.
Le backend (repo séparé) porte le moteur, les règles et la persistence; ce repo ne fait que présenter l'interface et consommer l'API REST.

### MVP (cible actuelle)

- Connexion Auth0 pour 8 utilisateurs.
- Navigation principale: `gameplan`, `players`, `teams`, `training`, `calendar`, `match-summary`.
- Affichage fiable des données backend; aucun calcul métier côté front.

## Architecture du repo (Angular)

- `src/app`:
  - `calendar`, `gameplan`, `login`, `mainpage`, `match-summary`, `players`, `teams`, `training`: features/pages.
  - `models`: types/DTO TypeScript.
  - `utils`: helpers UI/formatage.
  - `session.store.ts`: état applicatif léger (signals).
  - `app.config.ts`: providers (router, http, Auth0).
  - `app.routes.ts`: routes + guards.
- `src/styles.css`: styles globaux.
- `src/material-theme.scss`: thème Angular Material.

### Séparation stricte (intention)

- Les composants restent "minces": affichage + orchestration UI.
- La logique d'accès API vit dans des services/store, pas dans les templates.
- Les modèles TypeScript dans `models` reflètent l'API backend (pas d'inventions métier côté front).

## API backend


- Base prod : 'https://allauchsimulator.onrender.com'

- Base locale: `http://localhost:8080`.
- CORS attendu: `http://localhost:4201` (frontend).
- Intercepteur Auth0: token injecté sur `http://localhost:8080/*`.

## Auth / sécurité

- Auth0 Angular SDK configuré dans `app.config.ts`.
- `AuthGuard` sur les routes sensibles dans `app.routes.ts`.
- L'UI n'implémente pas de règles d'accès complexes: elle reflète l'état backend.

## UI / styles

- Angular Material + Tailwind + DaisyUI sont disponibles.
- Thème principal dans `src/material-theme.scss`.
- Styles globaux dans `src/styles.css`.

## Commandes utiles (repo)

- Lancer le front: `npm run start` (ou `ng serve`)
- Build: `npm run build`
- Tests: `npm run test`

## Notes d'outillage (workspace)

- `rg` (ripgrep) n'est pas disponible; utiliser PowerShell `Select-String` pour les recherches.

## Workflow de l'agent (a appliquer a chaque tache)

1. Reformuler le besoin en 1-2 phrases.
2. Proposer une solution simple (pas d'overengineering).
3. Implementer en respectant la séparation UI / services / modèles.
4. Ajouter/adopter les tests.
5. Verifier: formatage, gestion d'erreurs, accessibilité de base, performances.
6. Livrer: code + courte explication + hypotheses/risques.

## Guidelines techniques (code)

### Angular/HTML modernes

- Viser les derniÃ¨res versions d'Angular et des standards HTML.
- Ã‰viter `*ngFor` et `*ngIf` quand c'est possible; prÃ©fÃ©rer les Ã©quivalents modernes (`@for`, `@if`, `@switch`, `@defer`).

### Architecture front

- Composants: presentational + orchestration UI, pas de logique métier.
- Services/Stores: appels API + mapping vers modèles UI.
- Modèles: TypeScript strict, pas de `any` dans les payloads API.

### Data fetching / state

- Préférer `HttpClient` + RxJS (`map`, `tap`, `catchError`).
- Éviter les `subscribe` manuels dans les composants; privilégier `async` pipe ou signals.
- État global minimal: utiliser `SessionStore` pour l'identité/club.

### Erreurs & UX

- En cas d'erreur API, montrer un message stable et exploitable (pas de stack brute).
- Ne pas masquer une erreur logique par une "valeur par défaut".

### Performance

- Utiliser `trackBy` dans les `*ngFor` volumineux.
- Éviter les recalculs lourds dans les templates.

### Tests

- Composants: tests de rendu et d'interaction sur les comportements clés.
- Services: `HttpTestingController` pour valider les calls API.

### Pratiques de code (règles explicites)

- Pas de logique métier qui duplique le backend.
- Pas de "clamp"/normalisation silencieuse d'inputs: si invalide, faire échouer clairement.
- Garder les modèles alignés avec l'API backend (types, champs, nullability).

---
> Source: [Sanguiwara/allauch-basketball-simulator-frontend](https://github.com/Sanguiwara/allauch-basketball-simulator-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
