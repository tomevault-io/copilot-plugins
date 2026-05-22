## flashbackmirror

> | Produit | **Flashback** — app web d’enregistrement / buffer / flashback d’entraînement, **statique**, sans bundler |

# Flashback Mirror — contexte pour l’agent

## Référence rapide (projet)

| Élément | Détail |
|--------|--------|
| Produit | **Flashback** — app web d’enregistrement / buffer / flashback d’entraînement, **statique**, sans bundler |
| Code servi en prod | `docs/index.html`, `docs/script.js`, `docs/styles.css` |
| Doc produit | `Description docs/backlog.md`, `Description docs/requirements-user-stories.md` (quoter le chemin en shell : espace dans le nom du dossier) |
| Déploiement | GitHub Pages : branche **`main`**, dossier **`/docs`** |
| URL publique | https://samuelpilot86.github.io/flashbackmirror/ |
| Mise à jour du site | `git commit` + `git push` des changements sous `docs/` (délai côté GitHub) |
| Source de code | Travailler **directement dans `docs/`** — le dossier `App/` n’existe pas dans ce dépôt |
| Assets | `Logo/` à la racine ; référencer depuis `docs/` en chemins relatifs si besoin |
| Test local | Servir `docs/` en HTTP (ex. `npx --yes serve docs` depuis la racine), pas seulement `file://` (caméra / APIs) |

## Git — fichiers à ne pas versionner

Ces motifs sont dans **`.gitignore`** : ne pas les ajouter au dépôt ni proposer de les commiter.

- `docs/index *.html` — copies locales de secours de l’index (le fichier **`docs/index.html`** sans espace après `index` reste versionné).
- `docs/old/` — archives HTML locales.

Ne retirer ces règles du `.gitignore` qu’avec accord explicite du mainteneur.

## Conventions de travail (équipe / utilisateur)

- **Exécution** : environnement réel — lancer les commandes et vérifications soi-même quand c’est possible, plutôt que de se limiter à des instructions à copier-coller.
- **Résilience** : en cas d’échec, diagnostiquer, varier l’approche et réessayer plutôt qu’abandonner après une seule tentative.
- **Langue** : répondre à l’utilisateur en **français** ; style clair, phrases complètes (plutôt qu’une suite de fragments télégraphiques).
- **Périmètre** : modifier uniquement ce qui sert à la demande ; pas de refactors ni de fichiers hors sujet ; **ne pas** ajouter de fichiers Markdown de documentation non demandés.
- **Code** : lire le contexte autour des changements ; rester aligné sur le style, les abstractions et le niveau de commentaires existants ; ne pas supprimer du code ou des commentaires utiles hors du périmètre.
- **UI** : viser un rendu cohérent avec `styles.css` et l’existant (espacements, typo, couleurs).
- **Git** : messages de commit en phrases complètes, grammaire soignée, détail utile seulement (quand l’utilisateur demande un commit).
- **Conversation** : interpréter la demande dans le **fil** de la discussion (objectif implicite, raffinements), pas uniquement le dernier message isolé.
- **Citations** : pour parler du code déjà dans le dépôt, citer fichier et lignes quand c’est utile à la compréhension.

---
> Source: [samuelpilot86/flashbackmirror](https://github.com/samuelpilot86/flashbackmirror) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
