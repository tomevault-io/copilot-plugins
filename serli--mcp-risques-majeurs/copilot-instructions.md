## mcp-risques-majeurs

> Serveur MCP (Node.js / TypeScript) exposant des données sur les risques majeurs (climatiques et technologiques) en France, basé sur les APIs ouvertes de Géorisques et de l'IGN.

# MCP Risques Majeurs

Serveur MCP (Node.js / TypeScript) exposant des données sur les risques majeurs (climatiques et technologiques) en France, basé sur les APIs ouvertes de Géorisques et de l'IGN.

## Objectif

Projet d'apprentissage MCP, progression step-by-step.

## Structure du projet

Chaque step est un dossier autonome avec son propre code, `package.json` et `README.md` :

```
├── README.md                  ← point d'entrée du workshop
├── typescript-basics.md       ← rappel TypeScript + Zod
├── CLAUDE.md                  ← contexte pour Claude (ce fichier)
├── step-1/                    ← Step 1 : Hello MCP
│   ├── README.md
│   ├── package.json
│   ├── tsconfig.json
│   └── src/index.ts
├── step-2/                    ← Step 2 (à venir)
│   └── ...
└── ...
```

## Commandes (par step)

```bash
cd step-X           # se placer dans le step voulu
npm install         # installer les dépendances
npm run build       # compiler (tsc)
npm run dev         # compiler en mode watch
node dist/index.js  # lancer le serveur
```

## APIs utilisées

### API Géorisques V1
- **Base URL :** `https://georisques.gouv.fr/api/v1`
- **Auth :** Aucune
- **Pagination :** `page` (1-indexed) + `page_size` (default 10)
- **Paramètres de localisation :** `latlon` (format `"longitude,latitude"`), `code_insee`, `rayon` (max 10000m)
- **Endpoints exclusifs V1 :** radon, cavités, catnat, TRI, AZI, PAPI, rapport risque (JSON + PDF)

### API Géorisques V2
- **Base URL :** `https://georisques.gouv.fr/api/v2`
- **Auth :** Bearer token requis (`Authorization: Bearer <token>`)
- **Pagination :** `pageNumber` (0-indexed) + `pageSize` (default 10, max 1000)
- **Paramètres de localisation :** `longitude`, `latitude` (séparés), `codesInsee`, `rayon` (max 20000m)
- **Avantages V2 :** recherche par parcelle cadastrale, géométrie WKT, téléchargement de documents PPR/ICPE

### API Géocodage IGN (Geoplateforme)
- **Base URL :** `https://data.geopf.fr/geocodage`
- **Auth :** Aucune
- **Endpoints clés :** `/search` (géocodage), `/reverse` (géocodage inverse)
- **Retour :** GeoJSON FeatureCollection avec `citycode` (= code INSEE), coordonnées, label, etc.

## Plan de progression

### Step 1 — Hello MCP ✅
- [x] Initialiser le projet (npm, TypeScript, `@modelcontextprotocol/sdk`)
- [x] Créer un serveur MCP minimal avec un tool `geocode`
- [x] Tool `geocode` : appelle l'API IGN `/search` pour transformer une adresse en coordonnées + code INSEE
- [x] Tester avec Claude Desktop ou Claude Code
- **Apprentissages :** structure serveur MCP, tools, transport Streamable HTTP

### Step 2 — Premier outil de risques
- [ ] Tool `get_risques_commune` : appelle `/api/v1/gaspar/risques` (liste tous les risques d'une commune)
- [ ] Chaînage géocodeur → risques (adresse → code INSEE → risques)
- **Apprentissages :** enchaînement d'appels API, descriptions riches de tools

### Step 3 — Risques naturels détaillés
- [ ] Tools : `get_seisme`, `get_argiles`, `get_radon`, `get_cavites`, `get_catnat`
- [ ] Utiliser V1 pour radon/cavités/catnat, V2 pour sismique/argiles
- **Apprentissages :** gérer V1 vs V2, gestion de token V2

### Step 4 — Risques technologiques
- [ ] Tools : `get_installations_classees` (ICPE/SEVESO), `get_installations_nucleaires`, `get_sites_pollues`
- [ ] Recherche par rayon autour d'un point
- **Apprentissages :** paramètres avancés, pagination

### Step 5 — Plans de prévention (PPR)
- [ ] Tools : `get_pprn` (naturels), `get_pprt` (technologiques), `get_pprm` (miniers)
- [ ] Filtrage par type d'aléa
- **Apprentissages :** filtres complexes, codes de référence

### Step 6 — MCP Resources
- [ ] Exposer des resources statiques : référentiels (familles de risques, codes aléas, états PPR)
- **Apprentissages :** différence Tools vs Resources

### Step 7 — Resource Templates & rapport PDF
- [ ] Resource template `risques://commune/{code_insee}/rapport`
- [ ] Récupération du rapport PDF via `/api/v1/rapport_pdf`
- **Apprentissages :** URI templates, content types variés

### Step 8 (bonus) — MCP Apps / UI Resources
- [ ] Visualisations : cartes de zones de risques
- **Apprentissages :** possibilités UI de MCP

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Serli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
