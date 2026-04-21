## bibmaths

> LibMaths est un site de cours et d'annales de concours pour etudiants en CPGE (classes preparatoires). Le public cible a un **niveau faible a moyen** — les corrections doivent etre accessibles, pas ecrites "par des profs pour des profs".

# LibMaths — Consignes pour les corrections pedagogiques

## Contexte du projet

LibMaths est un site de cours et d'annales de concours pour etudiants en CPGE (classes preparatoires). Le public cible a un **niveau faible a moyen** — les corrections doivent etre accessibles, pas ecrites "par des profs pour des profs".

## Philosophie des corrections

### Probleme a resoudre

Les corrections classiques supposent que l'etudiant connait deja le cours, voit les astuces, et comprend pourquoi on fait telle manipulation. Pour un niveau faible, c'est comme lire la solution sans comprendre la demarche.

### Principe fondamental

Une correction classique dit **quoi faire**. Nos corrections expliquent **pourquoi on le fait**, **comment y penser**, et **ou les autres se trompent**. L'objectif n'est pas juste de comprendre un exercice, mais d'acquerir les reflexes pour les exercices similaires.

### Les 3 couches de lecture

Chaque question doit etre traitee en 3 couches :

1. **"Qu'est-ce qu'on me demande ?"** — Reformuler l'enonce en francais simple. Pas de jargon inutile.
2. **"Pourquoi on fait ca ?"** — La motivation derriere chaque etape. Pas juste "on prend le rotationnel", mais "on veut eliminer B pour obtenir une equation sur E seul — c'est comme resoudre un systeme de 2 equations a 2 inconnues".
3. **"Comment on fait ?"** — Le calcul etape par etape, avec les resultats encadres.

### Les 5 types d'encadres

Chaque correction utilise 5 types d'encadres visuels distincts :

| Type | Couleur | Classe CSS | Role | Quand l'utiliser |
|------|---------|-----------|------|-----------------|
| Rappel de cours | Bleu | `.callout--rappel` | Reposer les fondations avant d'attaquer | Avant chaque question qui necessite un theoreme ou une definition |
| "Pourquoi ?" | Orange | `.callout--pourquoi` | L'intuition, l'idee directrice, l'analogie | Avant ou pendant une etape non evidente |
| Resultat | Dore | `.callout--resultat` | La formule finale encadree | A la fin de chaque question, le resultat a retenir |
| Piege | Rouge | `.callout--piege` | L'erreur frequente au concours | Quand le rapport de jury signale une erreur recurrente |
| Methode | Vert | `.callout--methode` | La recette reutilisable | Quand la technique est transposable a d'autres exercices |

### Regles de redaction

1. **Exemples concrets avant la theorie** — Toujours montrer un tableau de valeurs numeriques ou un cas particulier avant la formule generale. Ex : montrer X=1,2,3,4 -> Y=1,1,2,2 avant d'ecrire P(Y=k).

2. **Analogies physiques** — Utiliser des images du quotidien. "Comme une balle qui rebondit sur un mur" pour la pression de radiation. "Comme lancer une piece truquee" pour la loi geometrique.

3. **Progression par difficulte** — Utiliser les badges Standard (vert) / Intermediaire (orange) / Avance (rouge). Un niveau faible doit maitriser les parties Standard, tenter les Intermediaire, ne pas paniquer sur les Avance.

4. **Chaque etape justifiee** — Jamais de "il est clair que", "on verifie aisement", "il suffit de". Si c'est une etape, on la montre. Si c'est long, on la met dans un toggle `<details>`.

5. **Les pieges du jury** — Les rapports de jury disent exactement ou les candidats se trompent. Signaler ces erreurs dans des encadres rouges "Piege classique".

6. **Toggles pour les demonstrations longues** — Les preuves optionnelles ou les verifications sont dans des blocs `<details>` pour ne pas surcharger. Titre du toggle : "Voir la preuve de..." ou "Verification detaillee".

7. **LaTeX soigne** — Utiliser KaTeX. Formules inline `$...$` pour le texte, formules display `$$...$$` pour les resultats importants. Les resultats finaux sont encadres avec `\boxed{}`.

## Structure technique

### Systeme d'annales (3 couches)

1. **`data/exercices.json`** — Index leger. Chaque reference : banque (b), epreuve (e), partie (p), questions (q), difficulte (d).
2. **`data/corrections/*.json`** — 50 fichiers, un par chapitre. Corrections question par question en LaTeX.
3. **`data/enonces/*.json`** ou `cours/annales/*.html` — Enonces structures avec ancres par question.

### Banques de concours

| Code | Nom complet | Slug URL | Couleur |
|------|------------|----------|---------|
| C | CCINP | ccinp | Bleu #4A90D9 |
| CS | Centrale-Supelec | centrale | Orange #D4873B |
| M | Mines-Ponts | mines | Vert #4CAF50 |
| X | X-ENS | xens | Violet #9C27B0 |
| E | E3A-Polytech | e3a | Rose #E91E63 |

### Format d'une reference dans exercices.json

```json
{"b": "E", "e": "Maths 1 2024", "p": "Exercice 1", "q": "3 q. : Q1, Q2, Q3", "d": "S"}
```

### Format d'un bloc de correction

```json
{
  "banque": "E3A-Polytech",
  "epreuve": "Maths 1 2024",
  "partie": "Exercice 1",
  "questions": "3 q. : Q1, Q2, Q3",
  "difficulte": "Standard",
  "intitule": "Description + ce qu'il faut savoir faire",
  "corrections": [
    {"question": "Q1", "correction": "Texte LaTeX detaille..."}
  ]
}
```

### Pages de correction standalone

Les pages de correction detaillees (comme `e3a-2024-correction.html`) sont des pages HTML autonomes avec tous les styles inline. Elles utilisent le template des pages d'annales (`cours/annales/`) avec les styles supplementaires pour les encadres pedagogiques.

## Priorites pedagogiques (P1/P2/P3)

Pour les annales E3A-Polytech MP, les exercices sont classes en priorite :

- **P1** — Exercice tres classique, a savoir refaire les yeux fermes. A transformer en references "a savoir faire".
- **P2** — Exercice important mais moins central ou plus technique.
- **P3** — Exercice utile pour enrichir la base plus tard.

Le rapport fusionne `data/e3a-polytech-mp-fusionne.md` contient le detail par exercice avec les priorites, les methodes cles, et les "ce qu'il faut savoir faire".

## Systeme "Mon Parcours" — Correction et suivi des essais

### Declenchement

Quand l'utilisateur envoie une **photo/scan de copie manuscrite** avec une indication de l'exercice (ex: "E3A 2024 exo 1"), Claude doit :

1. **Lire la photo** et transcrire ce que l'etudiant a ecrit
2. **Identifier l'exercice** (sujet, annee, numero)
3. **Corriger question par question** selon le bareme standardise
4. **Evaluer /20** avec la formule lineaire
5. **Creer le fichier JSON** dans `data/parcours/AAAA-MM-JJ_sujet-exoN.json`
6. **Mettre a jour** `data/parcours/profil.json`
7. **Donner la correction** dans le chat avec commentaires et conseils

### Bareme standardise — Notation lineaire

Chaque question est notee sur 4 points max :

| Critere | Points | Description |
|---------|--------|-------------|
| Resultat correct | 1 pt | La reponse finale est juste |
| Methode/raisonnement | 1 pt | La demarche est correcte et adaptee |
| Justification/rigueur | 1 pt | Hypotheses citees, theoremes nommes, implications justifiees |
| Redaction | 1 pt | Phrases claires, notations propres, logique visible |

Ajustements :
- Question de cours : /2 (resultat 1pt + rigueur 1pt)
- Question longue (demo multi-etapes) : /6 (resultat 1pt + methode 2pt + justification 2pt + redaction 1pt)
- Question calculatoire simple : /3 (resultat 1pt + methode 1pt + calcul exact 1pt)

Note finale : `note = (total_points / total_points_max) * 20`, arrondi a 0.5 pres.

### Types de commentaires (3 niveaux)

**Niveau 1 — Par question :**
- "V Resultat correct, methode adaptee"
- "X Tu as trouve le bon resultat mais sans justifier la convergence"
- "~ Bonne idee de passer aux complexes mais erreur de calcul etape 3"

**Niveau 2 — Bilan de l'exercice :**
- Forces : "Tu maitrises le calcul de lois discretes"
- Faiblesses : "Les justifications de convergence sont systematiquement absentes"

**Niveau 3 — Conseils concrets :**
- "Avant de calculer une esperance, ecris TOUJOURS : la serie converge absolument car..."
- "Entraine-toi sur 3 exercices de series entieres avec rayon a determiner"

### Categories d'erreurs standardisees

| Categorie | Code | Exemples |
|-----------|------|----------|
| Convergence | CONV | Oubli de justifier convergence, mauvais critere |
| Rigueur | RIG | "Il est clair que", hypotheses non verifiees |
| Calcul | CALC | Erreur de signe, facteur oublie |
| Methode | METH | Mauvaise approche, outil inadapte |
| Redaction | RED | Notations incoherentes, manque de structure |
| Theoreme | TH | Nom du theoreme absent, conditions non verifiees |
| Symetrie/Astuce | SYM | Calcul frontal au lieu d'exploiter une symetrie |
| Cas particulier | CAS | Oubli d'un cas (n=0, x=0, matrice nulle...) |

### Format du fichier essai JSON

```json
{
  "date": "2026-04-03",
  "sujet": "E3A-Polytech MP 2024",
  "exercice": "Exercice 1",
  "theme": "Probabilites -- Loi geometrique",
  "chapitres": ["probabilites_spe"],
  "note": 14,
  "noteMax": 20,
  "questions": [
    {
      "numero": "Q1",
      "reussi": true,
      "note": 4,
      "noteMax": 4,
      "commentaire": "Correct. La verification sum pq^k = 1 est bien faite.",
      "erreurs": [],
      "codesErreurs": []
    }
  ],
  "bilanGeneral": "...",
  "erreursRecurrentes": ["..."],
  "conseilsPourLaSuite": "..."
}
```

### Page Mon Parcours

La page `parcours.html` affiche le dashboard de progression avec :
- Note moyenne, nombre d'essais, courbe de progression
- Top erreurs recurrentes
- Progression par chapitre
- Historique chronologique des essais
- Conseils personnalises

Les donnees sont chargees depuis `data/parcours/profil.json` et les fichiers d'essais individuels.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atimoz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
