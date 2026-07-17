## appintervenants

> Projet conforme au CLAUDE.md global : HTML / CSS minimaliste écrits main, JS Vanilla DOM-local, servis en fichiers statiques bruts sur GitHub Pages. **Aucun build navigateur, aucun bundler, aucun framework, aucune dépendance runtime.**

# AppIntervenants — gouvernance

Projet conforme au CLAUDE.md global : HTML / CSS minimaliste écrits main, JS Vanilla DOM-local, servis en fichiers statiques bruts sur GitHub Pages. **Aucun build navigateur, aucun bundler, aucun framework, aucune dépendance runtime.**

## Conforme au global
- HTML + CSS écrits main (`styles.css`, classes sémantiques, dark mode en `prefers-color-scheme`).
- JS Vanilla en modules ES natifs (`src/*.js`), chargés directement par le navigateur.
- Icônes en SVG inline (plus de Lucide). Partiels d'onglets chargés par `fetch()`.
- PWA : `sw.js` et manifests écrits main (plus de vite-plugin-pwa).
- Déploiement : la source EST le déployé. La CI assemble `_site/` (copie de fichiers, zéro build).

## Exception, réduite à un seul fichier
Node / npm sont conservés **uniquement** pour `vitest` + `tsc --checkJs` sur le module de calcul salaire (`src/calculator.js`, annoté JSDoc, ~45 tests).

Justification : ce module calcule des salaires. On garde la garantie « le code livré au navigateur est exactement celui couvert par les tests » (fichier unique, importé par le navigateur ET testé). C'est la seule chose que le global ne peut pas fournir pour un calcul interactif côté client.

Hors ce module : rien d'autre n'introduit de Node, de build ou de dépendance.

## Levé du global, et pourquoi
- **Python / uv** : sans objet, PWA 100 % client sans runtime serveur.
- **HTMX** : sans objet, site 100 % statique (aucun backend pour servir des fragments). On utilise l'allocation « JS Vanilla DOM-local » de la règle.
- **pytest** : les seuls tests portent sur le calculateur JS, lancés par Vitest.

## Commandes
- `npx vitest run` : tests (calculateur + intégrité des données).
- `npx tsc --noEmit` : vérification de types (JSDoc, `checkJs`).
- `npx eslint .` / `npx prettier` : lint et format.
- Servir en local : `python3 -m http.server` à la racine, ouvrir `index.html` (ajouter `?agence=loches` pour l'autre agence).

Note : le hook global interdit Node en local ; `vitest`/`tsc` tournent en CI (GitHub Actions). Pour les lancer en local, lever le hook pour ce projet.

---
> Source: [emmanuelperreau/AppIntervenants](https://github.com/emmanuelperreau/AppIntervenants) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
