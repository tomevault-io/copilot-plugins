## distance-calculator

> Application React de calcul de distance entre villes françaises et Monts (37260, Indre-et-Loire).

# CLAUDE.md - Instructions pour Claude Code

## Contexte du projet

Application React de calcul de distance entre villes françaises et Monts (37260, Indre-et-Loire).

**Fonctionnalités :**
1. Import d'un fichier CSV (code postal, ville)
2. Calcul de distance à vol d'oiseau vers Monts
3. Application de coefficients par tranches de distance
4. Export CSV avec distances et/ou coefficients

## Structure du projet

```
distance-calculator/
├── generate_postal_db.py   # Script Python - génère la base de codes postaux
├── src/
│   ├── App.jsx             # Composant principal React
│   ├── postal_coords.js    # Base de données générée (à créer)
│   ├── main.jsx
│   └── index.css
├── sample_cities.csv       # Fichier test
└── README.md
```

## Commandes importantes

### Setup initial
```bash
# Créer le projet Vite + React
npm create vite@latest . -- --template react
npm install

# Générer la base de codes postaux (~6300 entrées)
python generate_postal_db.py
```

### Développement
```bash
npm run dev      # Serveur de dev sur localhost:5173
npm run build    # Build production
npm run preview  # Preview du build
```

## Fichier postal_coords.js

Ce fichier doit être généré par `generate_postal_db.py`. Il contient :
- ~6300 codes postaux français
- Coordonnées GPS (latitude, longitude)
- Format compact : `{"75001": [48.8602, 2.3477], ...}`

**Si le script échoue**, les données sont disponibles sur :
- https://www.data.gouv.fr/fr/datasets/communes-de-france-base-des-codes-postaux/
- https://geo.api.gouv.fr/communes

## Points techniques

### Calcul de distance
- Formule de Haversine (distance orthodromique / à vol d'oiseau)
- PAS de distance routière (nécessiterait une API externe)

### Format CSV
- Séparateur : point-virgule (;) standard français
- Encodage : UTF-8 avec BOM
- Détection auto du séparateur en entrée (; ou ,)

### Coordonnées de référence
- Monts (37260) : lat 47.2833, lon 0.7833 (environ)
- Coordonnées exactes dans postal_coords.js après génération

## Tâches possibles

1. **Générer la base** : Exécuter `python generate_postal_db.py`
2. **Tester l'app** : `npm run dev` puis importer `sample_cities.csv`
3. **Améliorer l'UI** : Les styles sont dans App.jsx (inline styles)
4. **Ajouter des fonctionnalités** :
   - Tri par distance
   - Recherche/filtre
   - Graphiques de distribution
   - Export Excel (.xlsx)

## Notes

- L'app fonctionne 100% côté client (pas de backend)
- Pas d'appels API externes (base embarquée)
- Traitement instantané même pour 26k lignes
- Interface en français

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArnaudRinquin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
