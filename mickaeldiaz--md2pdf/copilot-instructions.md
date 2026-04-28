## md2pdf

> Ce fichier donne à Claude Code (et autres agents LLM) le contexte nécessaire

# CLAUDE.md — Guide pour Claude Code

Ce fichier donne à Claude Code (et autres agents LLM) le contexte nécessaire
pour travailler efficacement sur ce projet.

---

## Mission du projet

Convertir les notes Obsidian (`.md`) en PDF **sobres et professionnels**,
à la qualité d'un document LaTeX compilé. L'outil doit rester :

- **100 % local** (aucune API externe au runtime)
- **Simple à lancer** (CLI, un fichier ou un dossier en argument)
- **Fidèle à la syntaxe Obsidian** (callouts, wikilinks, embeds, math)
- **Typographiquement sobre** (noir sur blanc, hiérarchie claire, pas de fioritures)

---

## Stack

| Couche           | Choix                                 | Pourquoi                                        |
| ---------------- | ------------------------------------- | ----------------------------------------------- |
| Parsing Markdown | `markdown-it-py` + `mdit-py-plugins`  | Plugins matures : dollarmath, footnote, etc.    |
| Code highlight   | `Pygments` (server-side, classes CSS) | Pas besoin de JS dans le PDF                    |
| Math             | **KaTeX** (dans Chromium)             | Qualité LaTeX, rendu en navigateur via auto-render |
| HTML → PDF       | **Playwright (Chromium)**             | Seule option Python fiable sur Windows avec JS  |

Node.js n'est pas disponible dans l'environnement cible — **ne pas y migrer**.
WeasyPrint a été écarté : pas d'exécution JS → pas de KaTeX de qualité.

---

## Arborescence

```
md2pdf/
├── cli.py          # argparse, collecte des fichiers, options
├── converter.py    # orchestrateur MD → HTML → PDF
├── renderer.py     # configuration markdown-it + plugins + Pygments
├── obsidian.py     # préprocesseurs Obsidian (AVANT markdown-it)
├── assets.py       # téléchargement lazy de KaTeX (1er run)
└── templates/
    ├── document.html  # gabarit HTML (placeholders __FOO__)
    └── style.css      # CSS typographique (inline dans le HTML généré)
```

Le pipeline :
```
.md source
  → strip_frontmatter
  → obsidian.preprocess  (embeds, callouts, wikilinks, tags)
  → markdown-it render   (math, code, tables, footnotes)
  → build_html           (template + CSS + assets KaTeX)
  → Playwright           (chargement file://, attente KaTeX, pdf())
.pdf
```

---

## Conventions

### Préprocessing Obsidian (obsidian.py)

L'ordre compte :
1. `transform_embeds` — réécrit `![[img.png]]` → `![alt](file:///abs/path)`
2. `rewrite_inline_images` — réécrit `![alt](rel.png)` → chemins absolus
3. `transform_callouts` — remplace `> [!type]` blocks par `<div class="callout">`
4. `transform_wikilinks` — `[[note]]` → `<span class="wikilink">note</span>`
5. `transform_tags` — `#tag` → `<span class="tag">#tag</span>`

Les callouts rendent leur corps via un **second appel markdown-it** récursif
(`render_inner`) pour que le markdown interne (maths, listes, gras) soit interprété.

### Échappement HTML

Toujours utiliser `html.escape(text, quote=False)` dans les préprocesseurs.
Les `&#x27;` générés par `quote=True` sont re-échappés par markdown-it et
apparaissent en texte brut dans le PDF. **Piège vérifié et corrigé — ne pas réintroduire.**

### Typographer markdown-it

**Désactivé** (`typographer: False`) : les smartquotes corrompent les
attributs HTML des ancres de footnote. **Ne pas le réactiver sans test dédié.**

### Math rendering

- Le plugin `dollarmath` émet `<span class="math math-inline">\(...\)</span>`
  et `<div class="math math-display">\[...\]</div>` via `_math_renderer`
- KaTeX auto-render (chargé dans le template HTML) remplace `\(...\)` / `\[...\]`
  par du MathML/HTML rendu
- Après rendu, le script positionne `window.__katex_done = true` — Playwright
  attend ce flag avant d'imprimer le PDF

### Page breaks

Dans `style.css` :
- `h1..h6 { break-after: avoid; break-inside: avoid; }` — évite les titres orphelins
- `pre, table, .callout, .math-display, img { break-inside: avoid; }` — contenu groupé
- `@page { size: A4; margin: 22mm; }` — format par défaut

**Ne pas** ajouter `break-before: page` sur `h1` sauf sur demande explicite.

---

## Commandes utiles

```bash
# Setup (une seule fois)
pip install -r requirements.txt
python -m playwright install chromium

# Conversion
python -m md2pdf samples/demo.md
python -m md2pdf "C:\chemin avec espaces\note.md"   # guillemets obligatoires
python -m md2pdf vault/ -r --vault-root vault/      # dossier + résolution images
```

Tester une régression : toujours relancer sur [samples/demo.md](samples/demo.md)
qui couvre maths, code, tables, callouts, wikilinks, footnotes, tasklists.

---

## Pièges connus

1. **Chemins Windows avec espaces** : PowerShell splitte l'argument.
   L'utilisateur doit toujours entourer de `"..."`. L'erreur `Path does not exist` avec un chemin tronqué = ce cas.

2. **Assets KaTeX absents** : téléchargés au premier `convert_file`. Nécessite
   accès réseau la première fois. Ensuite, 100 % offline. Si échec, vérifier
   `md2pdf/vendor/katex/` et supprimer pour retenter.

3. **`html.escape` par défaut échappe les quotes** → corrompt le texte. Toujours
   `quote=False` dans les préprocesseurs Obsidian.

4. **Regex de callout avec `re.DOTALL`** → avale tout le document après le
   premier callout. Le flag correct est `(?m)` seul, avec `[^\n]*` pour les
   lignes du body (pas `.*`).

5. **Images non trouvées** : l'ordre de résolution est : dossier note →
   `attachments/` → `assets/` → `images/` → remontée vers `vault_root` → 
   `rglob` récursif dans le vault. Sans `--vault-root`, la remontée s'arrête
   à la racine du disque.

---

## GUI + packaging

- [md2pdf/gui.py](md2pdf/gui.py) — Tkinter + `tkinterdnd2` (drag-drop cross-platform).
  Fallback automatique sur file-picker si tkinterdnd2 absent.
- [md2pdf/__main__.py](md2pdf/__main__.py) — dispatch : `sys.argv == 1` → GUI,
  sinon CLI. Les deux modes partagent `converter.convert_file`.
- [build.py](build.py) — wrapper PyInstaller :
  - Pré-télécharge KaTeX (`assets.ensure_katex`)
  - Détecte `%LOCALAPPDATA%\ms-playwright` (Win) / `~/Library/Caches/ms-playwright` (Mac)
  - Bundle Chromium via `--add-data`
  - Injecte un runtime-hook qui force `PLAYWRIGHT_BROWSERS_PATH` vers le dossier embarqué
- **Pas de cross-compilation** : lancer `python build.py` sur chaque OS visé.

## Ce qu'il ne faut PAS faire

- ❌ Migrer vers Node.js (pas dispo dans l'env cible)
- ❌ Remplacer Playwright par WeasyPrint (pas de JS = pas de KaTeX)
- ❌ Charger KaTeX depuis un CDN au runtime (contrainte : 100 % local)
- ❌ Réactiver `typographer: True` sans test de régression footnotes
- ❌ Ajouter des emojis décoratifs dans les callouts (spec : style sobre)
- ❌ Introduire des dépendances lourdes (pandoc, LaTeX, GTK/Cairo)

---

## Idées d'évolutions possibles (si demandées)

- Watch mode (`--watch`) via `watchdog` pour recompiler à la sauvegarde
- Table des matières auto (`--toc`) en début de document
- Thème sombre optionnel (`--dark`)
- Export par dossier avec un fichier `index.pdf` concaténé
- Support des diagrammes Mermaid (nécessiterait d'intégrer `mermaid.js` côté navigateur)

Ne **pas** les implémenter tant que l'utilisateur ne les demande pas.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MickaelDiaz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
