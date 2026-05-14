## porte-buanderie

> Conception d'un mecanisme de porte-etagere entre cuisine et buanderie.

# Instructions pour Claude Code

## Contexte du projet

Conception d'un mecanisme de porte-etagere entre cuisine et buanderie.
Le probleme central: la porte est tres epaisse (355mm pour contenir des etageres)
dans une ouverture de 870mm. La diagonale (940mm) empeche un pivot simple.

La solution trouvee est un **4-bar linkage** avec porte trapezoidale et cadres.

## Fichiers cles

- `calculs/optimisation_documentee.py` : LE script principal. Contient TOUS les
  parametres (dimensions piece, porte, mecanisme) et l'optimiseur. Tres bien
  documente, lire ce fichier en premier pour comprendre le projet.

- `calculs/animate_resultat.py` : Genere l'animation GIF. Importe les parametres
  de `optimisation_documentee.py`. Apres chaque optimisation, copier les params
  du resultat dans ce script avant de regenerer.

- `freecad/assembly_porte.py` : Macro FreeCAD qui genere un Assembly complet
  avec toutes les pieces (platines, bras, porte, murs). Animation via
  `goto(pct)` / `animate()` dans la console Python de FreeCAD.
  Genere `exports/assembly_porte.FCStd`.

- `dessins/generate_plans_charnieres.py` : Genere les plans SVG cotes des
  pieces de quincaillerie (platines, bras, axes, nomenclature).

## Comment travailler

1. **Modifier les parametres** : editer le haut de `optimisation_documentee.py`
2. **Optimiser** : `python3 calculs/optimisation_documentee.py`
3. **Copier les resultats** dans `animate_resultat.py`, `assembly_porte.py`,
   `generate_plans_charnieres.py` (section parametres de chaque fichier)
4. **Generer l'animation GIF** : `python3 calculs/animate_resultat.py`
5. **Generer les plans SVG** : `python3 dessins/generate_plans_charnieres.py`
6. **Generer l'assembly FreeCAD** : `freecadcmd freecad/assembly_porte.py`

## Contraintes importantes a ne pas oublier

- **380mm max** derriere le mur droit (frigo), EXTRA_BACK = 0
- Les pivots mur doivent etre physiquement **sur ou dans le mur droit** (x: 870-1250)
- La porte est en position fermee a **y=0** (face cuisine) et pivote vers la buanderie
- Le **chanfrein a ete retire** (CHAMFER = 0), pas de modification des murs
- Le **SWEEP** est optimise comme 9eme variable (pas fixe)
- La clearance a step 0 (position fermee) est ignoree car la porte touche x=0 et y=0 par design
- **Clearance hardware**: les pivots doivent etre a au moins 30mm de la surface
  (mur ou porte) pour loger la platine (5mm) + demi-tube bras (20mm) + jeu (5mm).
  Cette contrainte est dans les BOUNDS de l'optimiseur (MIN_PIVOT_CLEARANCE).

## Design des charnieres

**Platines** (4 identiques en concept, profondeurs differentes):
- Tole 5mm pliee: fond (vis au mur/porte) + plat (tient le pivot) + cote (rigidifie)
- 2 plis a 90 deg, 1 seule soudure (cote <-> fond)
- Le pivot (boulon D16) est entierement dans le plat avec 15mm de matiere autour
- Pivots a/A boulonnes par-DESSUS (bras au-dessus du plat)
- Pivots b/B boulonnes par-DESSOUS (bras en-dessous du plat)
- Toutes les platines au meme Z -> separation bras = tole + tube = 30mm

**Bras** (2 types: long 584mm et court 425mm):
- Tubes rectangulaires 40x25x3mm, perces D18 aux 2 extremites
- Le boulon-pivot traverse directement le tube

## Probleme connu: resolution de simulation

L'optimiseur utilise N_STEPS=50 (rapide). L'animation utilise 400 pas (precis).
La clearance peut varier entre les deux. Augmenter N_STEPS a 100+ pour un
resultat plus fiable mais plus lent.

## Systeme de coordonnees (IMPORTANT)

Axe X INVERSE par rapport a l'intuition initiale:
- x=0 : cote GAUCHE (trailing, mur de 355mm)
- x=870 : cote DROIT (charnieres, mur de 425mm)
- y=0 : face cuisine
- y positif : vers buanderie

Ceci correspond a la vue depuis la cuisine (gauche=gauche, droite=droite).

## Environnement

- Python 3.14 avec venv dans `.venv/`
- FreeCAD 1.1.0 installe (`freecadcmd` pour headless), MCP via socket
- Pillow installe pour GIF
- Pas de git dans ce dossier

---
> Source: [gagalago/porte-buanderie](https://github.com/gagalago/porte-buanderie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
