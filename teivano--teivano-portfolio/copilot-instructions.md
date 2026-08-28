## teivano-portfolio

> - **Toujours `Read` direct** — jamais `Agent` pour lire un fichier

# CLAUDE.md — Règles du projet teivano-portfolio

## Économie de tokens — PRIORITÉ ABSOLUE

### Lecture de fichiers
- **Toujours `Read` direct** — jamais `Agent` pour lire un fichier
- Fichier > 25k tokens → lire par tranches : `offset` + `limit` 150 lignes
- Ne lire que ce qui est nécessaire à la tâche en cours

### Utilisation des outils
- `Agent` uniquement pour des explorations multi-fichiers complexes (> 5 fichiers inconnus)
- `Glob` ou `Grep` pour trouver un fichier ou un symbole précis
- Si un outil échoue → récupérer **immédiatement** avec l'alternative directe, sans échange intermédiaire

### Comportement en cas d'erreur
- Erreur d'outil → corriger et relancer dans le même message, sans commenter l'erreur
- Ne jamais faire d'échange vide ("je vais faire X", "je reprends...") → faire X directement
- Ne pas expliquer ce qu'on va faire si on peut le faire maintenant

### Gestion du contexte
- Si la conversation dépasse 10 échanges sans livrable → suggérer `/compact`
- Préférer une nouvelle conversation pour chaque tâche distincte

## Workflow de développement

### Avant de coder
1. Lire les fichiers concernés (`Read`)
2. Identifier les IDs DOM, fonctions globales, dépendances externes
3. Lister les risques (conflits CSS, JS, breaking changes)
4. Soumettre le plan — attendre validation avant d'écrire

### Règle du GO — non négociable
**Ne jamais écrire dans un fichier du projet avant un GO formel et explicite.**
- Un retour, un constat de bug, une capture d'écran, une question ou un
  « qu'en penses-tu ? » ne sont **pas** des GO.
- Décrire un problème n'autorise pas à le corriger : on diagnostique, on
  propose, on attend.
- Sont autorisés sans GO : lire, chercher, mesurer dans le navigateur,
  écrire dans le scratchpad, exposer un plan.
- Le GO se dit : « go », « vas-y », « fais-le », « code », ou une consigne
  d'action sans ambiguïté. En cas de doute → demander.
- `git commit` et `git push` demandent leur propre GO, distinct de celui
  de l'écriture.

### Modifications HTML/CSS/JS
- Ne jamais renommer un ID DOM utilisé par du JS existant
- Ne jamais supprimer une fonction appelée en `onclick` inline
- Tester mentalement les conflits `transform` / `transition` avant d'écrire
- Conserver les classes existantes si du JS les cible (`.gc`, `.revealed`, etc.)

### Commits
- Message structuré : `type: résumé court\n\nDétails bullet points`
- Types : `feat`, `fix`, `refactor`, `style`, `chore`
- Toujours ajouter `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>`

## Stack technique
- **Frontend** : HTML vanilla + CSS custom + anime.js v4
- **Carte** : MapLibre GL v4.7.1
- **APIs** : STAR open data, SNCF, BAN, Esri, NASA, IGN, API-Ninjas, data.economie.gouv.fr
- **Cache STAR** : localStorage TTL 90s (évite les 429)
- **Proxies Vercel** : `/api/trains`, `/api/cars`, `/api/motorcycles`

## Environnement & déploiement
- **Domaine** : teivano.fr
- **Git** : `github.com/teivano/teivano-portfolio`, branche unique `main` (pas de branche de dev).
- **Déploiement** : Vercel lié au repo (`.vercel/project.json`). **Un `git push` sur `main` déclenche le déploiement auto** — aucune commande Vercel à lancer.
- **Pas de build / bundler** : les `.html` à la racine sont servis directement. Une page = un fichier `<nom>.html`.

## Ajouter une page
1. Créer `<nom>.html` à la racine. En bas de `<body>`, inclure `<script src="/nav.js"></script>` (menu commun) ; ajouter `<script src="/version.js"></script>` si on affiche la version.
2. Déclarer la page dans le tableau `LINKS` de `nav.js` pour la faire apparaître dans le menu hamburger.
   - Page publique : `{ href:'/<nom>.html', url:'/<nom>.html', title:'…', desc:'…' }`.
   - Page protégée par code PIN : mettre `href:'#'`, `pin:'/<nom>.html'` (+ `url` pour marquer « page courante ») et `pinHash` = SHA-256 du code (cf. `pin-guard.js`, comme GR20 `prepa-gr20.html` et Koala `koala.html`).
3. Commit + `git push` sur `main` → live sur teivano.fr.

## Ce qu'il ne faut pas casser
- `gcOf(id)` cherche `.gc` comme ancêtre direct — ne pas imbriquer les `.gc`
- `window._crystalize`, `window._animateCounters`, `window._createMapRipple` — fonctions globales attendues par les loaders
- `setVehicleMode()`, `setFuelView()`, `renderVehicle()` — appelées en `onclick` inline
- Cache STAR : clés `star_metro`, `star_bus`, `star_velos`, `star_etat`, `star_parkings`, `star_events`

---
> Source: [teivano/teivano-portfolio](https://github.com/teivano/teivano-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
